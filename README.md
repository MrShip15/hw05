[![Build Status](https://travis-ci.org/MrShip15/hw05.svg?branch=master)](https://travis-ci.org/MrShip15/hw05)
[![Coverage Status](https://coveralls.io/repos/github/MrShip15/hw05/badge.svg?branch=master)](https://coveralls.io/github/MrShip15/hw05?branch=master)
## Homework V

### Задание
1. Создайте `CMakeList.txt` для библиотеки *banking*.
Настройка git-репозитория hw04 для работы
```sh
% git remote remove origin
% hub create
% git push -u origin master
```
Cоздание `CMakeLists.txt`
```sh
% cat >> CMakeLists.txt <<EOF
cmake_minimum_required(VERSION 3.10)
project(banking)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_library(account STATIC banking/Account.cpp)
target_include_directories(account
 PUBLIC \${CMAKE_CURRENT_SOURCE_DIR}/banking)

add_library(transaction STATIC banking/Transaction.cpp)
target_include_directories(account
 PUBLIC \${CMAKE_CURRENT_SOURCE_DIR}/banking)
 target_link_libraries(transaction account)
EOF
```
Подключение к репозиторию подмодуля **Google Test**, выбор версии с помощью переключения ветки
```sh
% mkdir third-party
# Клонирование репозитория Google к своему репозиторию как подмодуль(проект в проекте)
% git submodule add https://github.com/google/googletest third-party/gtest
Cloning into '/home/ivan/MrShip15/workspace/projects/hw05/third-party/gtest'...
remote: Enumerating objects: 20049, done.
remote: Total 20049 (delta 0), reused 0 (delta 0), pack-reused 20049
Receiving objects: 100% (20049/20049), 7.33 MiB | 1.01 MiB/s, done.
Resolving deltas: 100% (14816/14816), done.
% cd third-party/gtest && git checkout release-1.10.0 && cd ../..
% git add third-party/gtest
```
Модифицируем CMakeList.txt
```sh
# Вставка текста в файл после строки
# Добавление опции для сборки тестов
% sed -i "" '/set(CMAKE_CXX_STANDARD_REQUIRED ON)/a\
option(BUILD_TESTS "Build tests" OFF)
' CMakeLists.txt
# Вставка в конец файла
# Добавление сборки тестов
% cat >> CMakeLists.txt <<EOF
if(BUILD_TESTS)
  enable_testing()
  add_subdirectory(third-party/gtest)
  file(GLOB \${PROJECT_NAME}_TEST_SOURCES tests/*.cpp)
  add_executable(check \${\${PROJECT_NAME}_TEST_SOURCES})
  target_link_libraries(check account transaction gtest_main gmock_main)
  add_test(NAME check COMMAND check)
endif()
EOF
```
2. Создайте модульные тесты на классы `Transaction` и `Account`.
    * Используйте mock-объекты.
    * Покрытие кода должно составлять 100%.
Создание тестов для класса `Account`
```sh
% cat >> tests/test1.cpp <<EOF
#include <Account.h>
#include <gtest/gtest.h>
// Тест на проверку правильности конструктора
TEST(Account, Constructor)
{
Account a(5,400);

EXPECT_EQ(a.id(),5);
EXPECT_EQ(a.GetBalance(),400);
}
// Тест на проверку правильность изменения баланса
TEST(Account, ChangeBalance)
{
  Account a(5,400);
  a.Lock();
  a.ChangeBalance(100);
  EXPECT_EQ(a.GetBalance(),500);
}
// Тест на проверку состояния аккаунта(открыт/закрыт)
TEST(Account, Lock)
{
  Account a(5,400);
  a.Lock();
  a.ChangeBalance(100);
  a.Unlock();
  EXPECT_EQ(a.GetBalance(),500);
}
EOF
```
Создание тестов для класса `Transaction`  && применение mock-объектов
```sh
% cat >> tests/test2.cpp <<EOF
#include <Account.h>
#include <Transaction.h>
#include <gmock/gmock.h>
#include <gtest/gtest.h>
// Создание mock-класса
class MockAccount : public Account {
public:
  MockAccount(){};
  MOCK_METHOD(int, GetBalance, (), (const, override));
  MOCK_METHOD(int, id, (), (const));
};
// Пример использования mock-объектов
TEST(Transaction, MakeTransaction) {

  MockAccount from;
  MockAccount to;
  Transaction transaction1;

  // Установка поведения
  EXPECT_CALL(from, id()).WillOnce(testing::Return(1));
  EXPECT_CALL(from, GetBalance()).WillOnce(testing::Return(1000));
  EXPECT_CALL(to, id()).WillOnce(testing::Return(2));
  EXPECT_CALL(to, GetBalance()).WillOnce(testing::Return(100));
  EXPECT_TRUE(transaction1.Make(Account(from.id(), from.GetBalance()),
                                Account(to.id(), to.GetBalance()), 500));
}
// Проценты за перевод
TEST(Transaction, Fee){
  Transaction transaction1;
  EXPECT_EQ(transaction1.fee(),1);
  transaction1.set_fee(3);
  EXPECT_EQ(transaction1.fee(),3);
  EXPECT_TRUE(transaction1.Make(Account(3, 1000),
                                Account(4,100), 500));
}
EOF
```
Сборка проекта
```sh
# Генерация файлов для сборки с тестом
% cmake -H. -B_build -DBUILD_TESTS=ON
-- The C compiler identification is AppleClang 11.0.3.11030032
-- The CXX compiler identification is AppleClang 11.0.3.11030032
...
-- Build files have been written to: /home/ivan/MrShip15/workspace/projects/hw05/_build
########################################
% cmake --build _build
Scanning dependencies of target account
[  6%] Building CXX object CMakeFiles/account.dir/banking/Account.cpp.o
...
[100%] Linking CXX executable check
[100%] Built target check
$ cmake --build _build --target test
Running tests...
Test project /home/ivan/MrShip15/workspace/projects/hw05/_build
    Start 1: check
1/1 Test #1: check ............................   Passed    0.30 sec

100% tests passed, 0 tests failed out of 1

Total Test time (real) =   0.30 sec
```
Запуск тестов
```sh
% _build/check
Running main() from /home/ivan/MrShip15/workspace/projects/hw05/third-party/gtest/googletest/src/gtest_main.cc
[==========] Running 5 tests from 2 test suites.
[----------] Global test environment set-up.
[----------] 3 tests from Account
[ RUN      ] Account.Constructor
[       OK ] Account.Constructor (0 ms)
[ RUN      ] Account.ChangeBalance
[       OK ] Account.ChangeBalance (0 ms)
[ RUN      ] Account.Lock
[       OK ] Account.Lock (0 ms)
[----------] 3 tests from Account (0 ms total)

[----------] 2 tests from Transaction
[ RUN      ] Transaction.MakeTransaction
1 send to 2 $500
Balance 1 is 499
Balance 2 is 600
[       OK ] Transaction.MakeTransaction (1 ms)
[ RUN      ] Transaction.Fee
3 send to 4 $500
Balance 3 is 497
Balance 4 is 600
[       OK ] Transaction.Fee (0 ms)
[----------] 2 tests from Transaction (1 ms total)

[----------] Global test environment tear-down
[==========] 5 tests from 2 test suites ran. (1 ms total)
[  PASSED  ] 5 tests.
# Запуск тестов с подробным выводом информации
$ cmake --build _build --target test -- ARGS=--verbose
Running tests...
...
100% tests passed, 0 tests failed out of 1

Total Test time (real) =   0.01 sec
```
3. Настройте сборочную процедуру на **TravisCI**.
```sh
% cat > .travis.yml <<EOF
language: cpp
os:
  - osx
jobs:
  include:
  - name: "Link an test"
    script:
    - cmake -H. -B_build -DBUILD_TESTS=ON
    - cmake --build _build
    - cmake --build _build --target test
    - _build/check
    - cmake --build _build --target test -- ARGS=--verbose
addons:
  apt:
    sources:
      - george-edison55-precise-backports
    packages:
      - cmake
      - cmake-data
EOF
```
Проверка `.travis.yml` на ошибки
```sh
% travis lint
Hooray, .travis.yml looks valid :)
% travis login --auto
% travis enable
```
4. Настройте [Coveralls.io](https://coveralls.io/).
Обновление `CMakeLists.txt`
```sh
% sed -i "" '/add_executable(check ${${PROJECT_NAME}_TEST_SOURCES})/a\
target_compile_options(check PRIVATE --coverage)
target_link_libraries(check PRIVATE account transaction gtest_main gmock_main  --coverage)
' CMakeLists.txt
```
Перепишем сборочную процедуру на **TravisCI**.
```sh
% cat > .travis.yml <<EOF
language: cpp
os:
  - osx
jobs:
  include:
  - name: "Link an test"
    script:
    - cmake -H. -B_build -DBUILD_TESTS=ON
    - cmake --build _build
    - cmake --build _build --target test
    - _build/check
    - cmake --build _build --target test -- ARGS=--verbose
  - name: "Coveralls.io"
    before_install:
    - pyenv rehash
    - pip install cpp-coveralls
    - pyenv rehash
    script:
    - cmake -H. -B_build -DBUILD_TESTS=ON
    - cmake --build _build
    - ./_build/check
    after_success:
    - coveralls --root . -E ".*gtest.*" -E ".*CMakeFiles.*"

addons:
  apt:
    packages:
      - cmake
      - cmake-data

EOF
```
Проверка `.travis.yml` на ошибки
```sh
% travis lint
Hooray, .travis.yml looks valid :)
```
`add`, `commit`, `push`
```sh
% git add .
% git commit -m "make hw"
% git push origin master   
```
## Links

- [C++ CI: Travis, CMake, GTest, Coveralls & Appveyor](http://david-grs.github.io/cpp-clang-travis-cmake-gtest-coveralls-appveyor/)
- [Boost.Tests](http://www.boost.org/doc/libs/1_63_0/libs/test/doc/html/)
- [Catch](https://github.com/catchorg/Catch2)

```
Copyright (c) 2015-2020 The ISC Authors
```
