# makefile学习

* 学习笔记网址: https://seisman.github.io/how-to-write-makefile/introduction.html#id10
* 默认文件名: Makefile-
* 

* 基本规则

  ```makefile
  target : prerequisites ...
      command
  ```

  如果targe文件不存在或者prerequisites中有一个以上的文件比target文件要新的话，command所定义的命令就会被执行。

* 概览

* ```makefile
  # 定义变量
  objects = main.o kbd.o command.o display.o \
      insert.o search.o files.o utils.o
  
  edit : $(objects) # 使用变量
      cc -o edit $(objects)
  main.o : main.c defs.h
      cc -c main.c
  kbd.o : kbd.c defs.h command.h
      cc -c kbd.c
  command.o : command.c defs.h command.h
      cc -c command.c
  display.o : display.c defs.h buffer.h
      cc -c display.c
  insert.o : insert.c defs.h buffer.h
      cc -c insert.c
  search.o : search.c defs.h buffer.h
      cc -c search.c
  files.o : files.c defs.h buffer.h command.h
      cc -c files.c
  utils.o : utils.c defs.h
      cc -c utils.c    
      
  .PHONY : clean # 隐晦规则，表示clean是个伪目标文件
  clean :
      rm edit $(objects)
  ```