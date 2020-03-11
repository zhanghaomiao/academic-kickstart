---
title: "fortran programming"
date: 2019-04-15 16:59:45
tags:
    - Fortran
    - C
summary: Fortran 易混淆语法
toc: true
# categories: Learn
---
{{% toc %}}

## 模块创建显示接口

在模块中编译和调用程序时, 过程接口的所有细节对编译器都是有效的. 当调用程序时, 编译器可以自动检测过程调用中的参数个数, 类型, 是否参数是数组, 已经每个参数的 INTENT 属性. 在模块内, 编译过程和用USE关联访问过程具有一个显示接口 (explicit interface). Fortran 编译器清楚的知道过程每个参数的所有细节. 不在模块内的过程称为隐式接口 (implicit interface). Fortran 编译器调用程序时, 不知道这些过程的信息.

## 函数/子程序作为参数传递

在调用和被调用函数中，函数被声明为外部量时(external), 用户自定义函数才可以当做调用参数传递．　当参数表中的某个名字被声明为外部变量，相当于告诉编译器在参数表中传递的是独立的已编译的函数.

- 函数作为参数传递

  ```fortran
  program main
      implicit none
      interface  ! must specify the interface
          subroutine runfunc(fun, array)
              real*8 :: array(:, :)
              real*8, external :: fun
          end subroutine
      end interface
      real*8 :: array(3,3)
      real*8,external :: fun1
      array = 2d0
      call runfunc(fun1, array)
  end program
  
  subroutine runfunc(fun, array)
      implicit none
      interface  !must specify the interface
          real*8 function fun(array)
              real*8 :: array(:, :)
          end function
      end interface
      real*8 :: array(:, :)
      print*, fun(array)
  end subroutine
  
  real*8 function fun1(array)
      implicit none
      real*8 :: array(:, :)
      integer :: i, j
      fun1 = 0d0
      do i = 1, size(array, 1)
          do j = 1, size(array, 2)
              fun1 = fun1 + array(i,j)
          enddo
      enddo
  end function
  ```

- 子程序作为参数传递

  ```fortran
  program main
      implicit none
      integer :: n = 5
      external :: add, prod
      real*8 :: array(5) = (/1,2,3,4,5/)
      call passsub(add, array, n)
      call passsub(prod, array, n)
  end program
  
  subroutine passsub(sub, array, n)
      integer :: n
      external :: sub
      real*8 :: array(n)
      call sub(array, n)
  end subroutine
  
  subroutine add(array, n)
      integer :: n
      real*8 :: array(n)
      integer :: i
      real*8 :: sum = 0
  
      do i = 1, n
          sum = sum+ array(i)
      enddo
      print*, sum
  end subroutine
  
  subroutine prod(array, n)
      integer :: n
      real*8 :: array(n)
      integer :: i
      real*8 :: cuml = 1
      do i = 1, n
          cuml = cuml * array(i)
      enddo
      print*, cuml
  end subroutine
  ```

## 子程序传递多维数组

- 显示结构的形参数组 (explicit-shape dummy arrays)

```fortran
  subroutine process1(data1, data2, n, m)
  integer, intent(in) :: n, m
  real,intent(in),dimension(n,m) :: data1
  real,intent(out), dimension(n,m) :: data2
  data2 = 3 * data1
  end subroutine
```

- 不定结构形参数组 (assumed-shape dummy arrays)

数组中的每个下标都用冒号代替, 只有子程序或者函数具有显示接口时, 才能使用这种数组. 通常采用将子程序放入模块中, 然后在程序中使用该模块. 编译器可以从接口信息判断每个数组的大小和结构.

``` fortran
module test_module
contains
    subroutine process2(data1, data2)
    real, intent(in), dimension(:,:) :: data1
    real, intent(out), dimension(:, :) :: data2
    data2 = 3 * data1
    end subroutine
end module
```

## 关键字参数和可选参数

- 关键字参数
  如果过程接口是显示的, 可以改变参数表中调用参数的顺序, 可以用关键字参数(keyword arguments) 提供更强的灵活性

  ```fortran
  module procs
  contains
      real, function calc(first, second, third)
      implicit none
      real, intent(in) :: first, second, third
      calc = (fisrt - second) / third
      end function calc
  end module
  ```

  在调用时, 即可以用

```fortran
program test_keywords
implicit none
write(*, *) calc(3., 1., 2.)
write(*, *) calc(first=3., second=1., third=2.)
write(*, *) calc(3., third=2., second=1.)
```

- 可选参数

  通过在形参申明中加入 `optional` 属性, 指定可选参数
  `integer, intent(in), optioncal :: upper_limit`
  可选参数是否出现, 可以用 **fortran** 中的 `present` 来判断

```fortran
  module process
  contains
      subroutine extremes(a,n, maxval, pos_maxval)
          integer, intent(in) :: n
          real, intent(in), dimension(n) :: a
          real, intent(out), optional :: maxval
          integer, intent(out), optional :: pos_maxval
          integer :: i
          real :: real_max
          integer :: pos_max

          real_max = a(1)
          pos_max = 1
          do i = 2, n
              max: if (a(i) > real_max) then
                  real_max = a(i)
                  pos_max = i
              end if max
          enddo
          if (present(maxval)) then
              maxval = real_max
          endif
          if (present(pos_maxval)) then
              pos_maxval = pos_max
          endif
      end subroutine
  end module
```

## 过程接口和接口块

想要使用不定结构形参数组或者可选参数这类 Fortran 高级特性, 程序必须要有显示接口. 创建显示接口最简单的方法是将过程放在模块中, 但是将过程放在模块中, 在一些场合不可能. 如过程和函数是用早期的 fortran 语言编写, 或者外部函数库是用 C 或者其他语言编写, 这样将过程放在一个模块中并不可能.

当不能把过程放在模块中, fortran 允许在调用程序中定义一个接口块, 接口快指定了过程所有的接口特征, 编译器根据接口快中的信息执行一致性检查.  接口的一般形式

```fortran
interface
    interface_body_1
    inteface_body_2
    ...
end interface
```

使用接口时, 将它放在调用调用单元的最前面

```fortran
program interface_example
implicit none
interface
    subroutine sort(a,n)
    implicit none
    real, dimension(:), intent(inout) :: a
    integer, intent(in) :: n
    end sobroutine sort
end interface

real, dimension(6) :: array = (/1., 5., 3., 2., 6., 4.,/)
integer :: nvals = 6
call sort(n = nvals, a = array)
end program
```

- 当为大型旧版程序或者函数库创建接口时, 可以将它们放在调用一个模块中, 可以使用 Use 访问

  ```fortran
  module interface_def ! no contains statement
  interface
      subroutine sort(array, n)
      ...
      end surtoutine
      subroutine sort2(array, n)
      ...
      end subroutine
  end interface
  end module
  ```

- 接口是独立的作用域, 接口块中的形参变量必须单独声明, 即使这些变量已经在相关的作用域中已经被声明过了. Fortran 2003 含有 import 语句, 如果 import 语句出现在接口定义中, 那么import 语句中指明的变量会被导入, 不需要在接口中再次申明.  如果 import 语句中没有带变量, 所有变量都会被导入.

    > Named entities from the host scoping unit are not accessible in an interface body that is not a module procedure interface body. The IMPORT statement makes those entities accessible in such interface body by host association.

## Generic

使用通用接口块定义过程，过程之间通过不同类型的输入参数区分，可以处理不同类型的数据. 给　`interface` 加入一个通用名，那么在接口快定义的每个过程接口可以看作一个通过过程．通用过程中所有过程要么都是子程序，要么都是函数.

例如对于不同类型的数据进行排序，创建一个通用子程序 `sort` 实现排序，可以使用通用接口块

```fortran
interface sort
    subroutine sorti(array, nvals)
    ...
    end subroutine sorti
    subroutine sortr(array, nvals)
    ...
    end subroutine sortr
    ...
    subroutine
end interface
```

## 模块中用于过程的通用接口

如果子程序都在一个模块中，并且已经有显示接口，如果再加入过程接口，这样是非法的，不能用上述方式添加通用接口．`fortran` 包含了一个可用在通用接口模块中的特殊`module procedure` 语句．

如果４个排序子程序定义在一个模块中，　子程序sort 通用接口是

```fortran
interface sort
    module procedure sorti
    module procedure sortr
    module procedure sortd
    ...
end interface sort
```

例如，找出一个数组的最大值和最大值位置，不管数组的类型

```fortran
module generic_maxval
implicit none
interface maxval
    module procedure maxval_i
    module procedure maxval_r
    ...
end interface
contains
    subroutine maxval_i (array, nvals, value_max, pos_maxval)
        implicit none
        integer, intent(in) :: nvals
        integer, intent(in), dimension(nvals) :: array
        integer, intent(out) :: value_max
        integer, intent(out), optional :: pos_maxval
        integer :: i
        integer :: pos_max
        value_max = array(i)
        pos_max = 1
        do i = 2, nvals
            if (array(i) > value_max) then
                value_max = array(i)
                pos_max = i
            end if
        enddo
        if (present(pos_maxval)) then
            pos_maxval = pos_max
        end if
    end sobroutine maxval_i

    subroutine maxval_r(array, nvals, value_max, pos_maxval)
    ...
    end subroutine

    ...
end module generic_maxval
```

## 内置模块

ortran 2003 定义了内置模块(intrinsic module), 这种模块是由 fortran 编译器创造者预定义和编写的．fortran　2003 中，有三种内置模块

1. `ISO_FORTRAN_ENV` 定义了描述特定计算机中存储器特征的向量(标准的整型包含多少bit, 标准字符包含多少bit),以及该机器所定义的I/O 单元
2. `ISO_C_BINDING` 包含了FORTRAN 编译其和特定处理器的c 语言互操作是必要数据
3. `IEEE` 处理器运行浮点数的特征

## Fortran 指针

- Fortran 申明变量为指针时, 需要使用 `Pointer` 属性, 并申明指针的类型
- 在 Fortran 变量的类型定义语句中, 加入`Target`属性, 将其声明为目标变量

```fortran
Program test_ptr
implicit none
real, pointer::p
real, target :: t1 = 10, t2=-17
p => t1
print*, p, t1, t2
p => t2
print*, p, t1, t2
end program
```

- 指针有三种状态, **undefined**, **associated**, **disassociated**, 通过使用 `NuLLIFY` 语句将指针和所有目标变量断开
- 指针数组. 指向数组的指针必须声明其指向数组的类型和维数, 不需要申明每一维的宽度

<script src="https://gist.github.com/zhanghaomiao/2591d8f6fe65f84d4984131cbd553317.js"></script>

## Mixing programming with C/C++

### Prerequisite

- 在C中, 所有子程序都是函数, void 类型函数不返回值
- 在 fortran 中, 函数会传递一个返回值, 子程序不传递返回值
- Fortran 调用 C 函数时
  - 被调用的C函数返回一个值, 会从Fortran 中将其作为函数来调用
  - 被调用的C函数不返回值, 将其作为子程序来调用
- C 函数调用 Fortran 子程序时
  - 如果被调用Fortran 子程序是一个函数, 会从 C 中将其作为一个返回兼容数据类型的函数来调用
  - 如果被调用的 Fortran 子程序是一个子例程, 会从 C 中将其作为一个返回 int 或 void 值的函数来调用. 如果Fortran 子例程使用交替返回 (返回值为 return 语句中的表达式), 会返回一个值. 如果 return 语句中没有出现表达式, 在Subroutine 语句中申明了交替返回, 则返回 0
- C 区分大小写, Fortran 忽略大小写
  - 在 C 子程序中, 使C函数名全部小写
  - 用 -U 选项编译 Fortran 程序, 会告知编译器保留函数/子程序名称现有大小写
- Fortran 在编译时,在子程序名和末尾加入下划线, C 编译时函数名称和用户指定名称一致. (Fortran 对于块过程名有两个前导下划线, 减少与用户指定子例程名的冲突)
  - 在 C 函数中, 通过在函数名末尾添加下划线更改名称
  - 使用 `BIND(C)` 属性声明表明外部函数 是C 语言函数
  - 使用 `-ext_names` 选项编译对于无下划线外部名称引用
- Fortran 传递字符型参数时, 会传递一个附加参数, 指定字符串的长度, 这个参数在 C 中为 `long int` 量, 按值进行传递
- C 数组从 0 开始, Fortran 数组默认以 1 开始
- C 数组按行主顺序存储,  Fortran 按列主顺序存储

### 数据类型兼容

- C 数据类型 `int, long int, long` 在 32 位环境下是等价的 (4 字节). 在 64 位环境中, `long` 和指针为8 字节
- 数组和结构的元素及字段需要兼容
- 不能按值传递数组, 字符串和结构
- 按值传递时, 可以使用 `%VAL(arg)`, 或者Fortran95 程序具有显含的接口块, 使用 `VALUE` 属性声明了伪参数
- 数据大小与对齐, 参考 https://docs.oracle.com/cd/E19205-01/819-5262/6n7bvdr18/

### 按引用传递参数

- 简单数据类型
- string
- structure
- array
- 按值传递参数

代码 example, 参考 <https://github.com/zhanghaomiao/C_FORTRAN_MIX>

### `ISO_C_BINDING` 模块

Fortran 2003 提供了内置模块,处理C 和  fortran 数据类型的转化

- `c_associated(c_ptr1[, c_ptr_2])`: determines the status of the C pointer c_ptr_1 or if c_ptr_1 is associated with the target c_ptr_2

```Fortran
subroutine association_test(a,b)
use iso_c_binding, only: c_associated, c_loc, c_ptr
implicit none
real, pointer :: a
type(c_ptr) :: b
if(c_associated(b, c_loc(a))) &
   stop 'b and a do not point to same target'
end subroutine association_test
```

- `c_f_pointer(cptr, fptr[,shape])` Assign the target, the C pointer, cptr to Frotran pointer.
- `c_f_procpointer(cptr, fptr)` assigns the target of the C function pointer `cptr` to the Fortran procedure pointer `fptr`
- `c_funloc(x)` determine the C address of the argument, return type is `c_funptr`
- `c_loc(x)` determine the C address of the argument, return type is `c_ptr`
- `c_sizeof(x)` calculate the number of bytes of storage the expression x occupies

```fortran
use iso_c_binding
real(c_float) :: r, s(5)
print *, (c_sizeof(s)/c_sizeof(r) == 5)
end
```

数据类型的对应关系,参考 [Fortran wiki](http://fortranwiki.org/fortran/show/iso_c_binding)

### Working with C++

- 在使用C++程序时, 需要加入 extern 'C' 关键字
- 在使用到 C++ 标准库的数据结构时, 链接时需要加入标准C++库 (`-lstdc++`)
- Example
    <script src="https://gist.github.com/zhanghaomiao/5c12cf5c54f405bf0b4df04c69ceb66c.js"></script>

## Reference

- http://fortranwiki.org/fortran/show/iso_c_binding
- https://gcc.gnu.org/onlinedocs/gfortran/Interoperability-with-C.html#Interoperability-with-C
- https://stackoverflow.com/questions/22813423/c-f-pointer-does-not-work
- http://www.yolinux.com/TUTORIALS/LinuxTutorialMixingFortranAndC.html
- https://docs.oracle.com/cd/E19205-01/819-5262/6n7bvdr18/
