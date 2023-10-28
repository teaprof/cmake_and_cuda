Как cmake ищет CUDA
===================

**REM**: Тут описан процесс поиска CUDA, если нужно быстрое решение, то можно сразу отмотать в конец текста.

Когда cmake встречает инструкцию `enable_language(CUDA)`, запускается цепочка скриптов для поиска компоненотов CUDA. К ним относятся

* исполняемые файлы, это прежде всего компилятор nvcc, компоновщики nvlink и fatbinary
* директории, в которых расположены header-файлы CUDA и библиотеки CUDA.

cmake ищет CUDA, следуя некоторым правилам. 

1. Сначала он проверяет "подсказки" - это переменные, переданные через командную строку, переменные из кэша `CMakeCache.txt` и переменные среды окружения. 
2. Потом он проверяет наличие компилятора nvcc, используя довольно общую утилиту из состава скриптов cmake, называемую `find_program`. Если `find_program` находит nvcc, то поиск прекращается и происходит переход к следующему этапу - определение расположения других компонентов CUDA по известному расположению компилятора.
3. Наконец, cmake проверяет стандартные пути (на линуксе это `/usr/local/cuda-X.Y`, именно сюда ставится CUDA, скачанная с сайта NVIDIA).

Когда компилятор уже найден, для поиска остальных компонентов CUDA, происходит запуск `nvcc -v fakefile`, где `fakefile` - имя любого несуществующего файла. Примерный вывод этой команды такой:

```console
tea@comp:~/projects/findcuda/build$ /usr/local/cuda/bin/nvcc -v fakefile
#$ _NVVM_BRANCH_=nvvm
#$ _SPACE_= 
#$ _CUDART_=cudart
#$ _HERE_=/usr/local/cuda/bin
#$ _THERE_=/usr/local/cuda/bin
#$ _TARGET_SIZE_=
#$ _TARGET_DIR_=
#$ _TARGET_DIR_=targets/x86_64-linux
#$ TOP=/usr/local/cuda/bin/..
#$ NVVMIR_LIBRARY_DIR=/usr/local/cuda/bin/../nvvm/libdevice
#$ LD_LIBRARY_PATH=/usr/local/cuda/bin/../lib:
#$ PATH=/usr/local/cuda/bin/../nvvm/bin:/usr/local/cuda/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/snap/bin
#$ INCLUDES="-I/usr/local/cuda/bin/../targets/x86_64-linux/include"  
#$ LIBRARIES=  "-L/usr/local/cuda/bin/../targets/x86_64-linux/lib/stubs" "-L/usr/local/cuda/bin/../targets/x86_64-linux/lib"
#$ CUDAFE_FLAGS=
#$ PTXAS_FLAGS=
nvcc fatal   : Don't know what to do with 'fakefile'
```
**Прим.**: Ключ `-v` означает 'verbose', а `fakefile` нужен только для того, чтобы инициировать процесс компиляции, в результате которого выводятся эти значения.

Для определения директории, куда установлена CUDA, cmake парсит вывод этой команды и находит строку, содержащую `TOP=<CUDA_ROOT>`.


Почему CUDA не находит cmake, если сделать символьную ссылку на компилятор
==========================================================================

При создании символьной ссылки `/usr/bin/nvcc`, указывающей на компилятор `/usr/local/cuda/bin/nvcc`, `find_program` находит именно её. На этом поиск завершается. Дальше cmake пытается найти недостающие компоненты CUDA путём вызова `cmake -v fakefile`. Вывод этой команды примерно такой:
```
tea@comp:~/projects/findcuda$ /usr/bin/nvcc -v fakefile
#$ _NVVM_BRANCH_=nvvm
#$ _SPACE_= 
#$ _CUDART_=cudart
#$ _HERE_=/usr/bin
#$ _THERE_=/usr/bin
#$ _TARGET_SIZE_=
#$ _TARGET_DIR_=
#$ _TARGET_SIZE_=64
nvcc fatal   : Don't know what to do with 'fakefile'
```

Видно, что даже сам компилятор не знает, где искать родные библиотеки CUDA (сравнить с предыдущим вызовом). Более того, попытка скомпилировать что-нибудь (эта техника используется cmake для определения параметров найденного компилятора) с помощью найденной ссылки на nvcc потребует явного указания include-директорий и директорий с библиотечными бинарными файлами, а cmake на данном этапе не знает, где их искать.

Даже если указать через -I и -L правильные пути, линкер nvlink всё ещё не найден. Можно его сделать system-wide (например, создать ссылку /usr/bin/nvlink), но не исключено, что это не решит полностью проблему поиска остальных компонентов cmake toolkit.

Как исправить
=============

Путь 1. Убрать символьную ссылку
------------------------
Убрать ссылку `/usr/bin/nvcc`. Тогда `enable_language(CUDA)` не найдёт CUDA в `/usr/local/cuda`. Почему так происходит - непонятно. Одно можно сказать точно, что при сравнении скриптов поиска CUDA в cmake 3.27 и 3.28 rc2 видны значительные изменения, что свидетельствует о том, что алгоритм поиска CUDA ещё не устоялся. Напротив, `findCUDAToolkit` как наследник `findCUDA` (объявлен deprecated), имеет устоявшийся алгоритм поиска CUDA. Поэтому файл `CMakeLists.txt` может содержать такое решение:
```CMakeLists.txt
cmake_minimum_required(VERSION 3.22)
project(prjname)

find_package(CUDAToolkit) # New in version 3.17
message(STATUS "CUDAToolkit_NVCC_EXECUTABLE = " ${CUDAToolkit_NVCC_EXECUTABLE})
set(CMAKE_CUDA_COMPILER ${CUDAToolkit_NVCC_EXECUTABLE})

enable_language(CUDA)
add_executable(aaa aaa.cu)
```



Путь 2. Через переменную окружения
----------------------------------

Задать переменную окружения

```console
export CUDACXX=/usr/local/cuda/bin/nvcc
```

При это в самом `CMakeLists.txt` путь к CUDA указывать не надо:

```CMakeLists.txt
cmake_minimum_required(VERSION 3.22) # VERSION should be not too small, for example, 3.14 or higher
project(prjname CUDA)
add_executable(aaa aaa.cu)
```

Тогда cmake найдёт nvcc по указанному пути, и, вызвав `nvcc -v fakefile`, вычислит все остальные зависимости.

Путь 3. Заменить символьную ссылку на простой скрипт
--------------------------------------------
Заменить символьную ссылку `/usr/bin/nvcc` на скрипт следующего содержания:

```bash
#!/bin/sh
exec /usr/local/cuda/bin/nvcc "$@"
```

В этом случае `nvcc -v fakefile` будет выдавать правильную информацию о расположении самого компилятора и всего CUDA Toolkit. Файл `CMakeLists.txt` будет выглядеть как в предыдущем пункте.

**Прим.**: именно так делается при установке ```nvidia-cuda-toolkit``` через `apt install`.<br/>

Замечания
=========

Команда `find_package(CUDAToolkit)` ищет pthreads, не совсем понятно, зачем. 

Скрипт findCUDAToolkit может выдавать warning, если какой-то компонент не найден. Например, может появиться такое сообщение
```console
CMake Warning at /snap/cmake/1336/share/cmake-3.27/Modules/FindCUDAToolkit.cmake:1072 (message):
  Could not find librt library, needed by CUDA::cudart_static
```
Оно означет, что статическая линковка с библиотеками CUDA не получится. Чтобы `find_package(CUDAToolkit)` работал корректно, надо `project(name)` объявлять до вызова `find_package(CUDAToolkit)`.

