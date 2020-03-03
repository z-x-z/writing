# 子程序

## show_str \ show_str_inverse

* parameter
  1. dh:  行号
  2.  dl:  列号
  3.  cl:  颜色值
  4.  ds:si:   字符串首地址

* function:

  ```asm
  code segment
          start:
              mov dh, 8
              mov dl, 3
              mov cl, 2
              mov ax, data
              mov ds, ax
              mov si, 0
              call init
  
              mov ax, 4c00h
              int 21h
          
          init:
          	;保护寄存器(子程序中所使用过的)
              push ax
              push dx
              push cl
              push ds
              push si
              push bx
              ;初始化相关参数(将bx置为相应的偏移地址=160*行号+2*列号)
              mov al, 160
              mul dh
              mov bx, ax
              ;由于2*列号可能超过8位，所以使用ax来存储
              mov ah, 0
              mov al, dl
              add ax, ax
              add bx, ax
              ;初始化目标段寄存器
              mov ax, 0b800h
              mov es, ax
          show_str:
          	;保存cl中的颜色样式值
          	mov ah, cl
          	;移动Ascii码（字节）至cl
              mov ch, 0
              mov cl, ds:[si]
              inc si
              ;检验cl是否为0
              jcxz ok
              ;输出Ascii码
              mov es:[bx], cl
              ;输出颜色属性
              mov es:[bx+1], ah
              mov cl, ah
              add bx, 2
          jmp short show_str
  			
          return:
          	;恢复现场
              pop bx
              pop si
              pop ds
           	pop cl
              pop dx
              pop ax
              ;返回主程序
              ret
        code ends
  ```



## divdw

* parameter:

  1. ax: dword型数据的低位
  2. dx: dword型数据的高位
  3. cx: 除数

* return:

  1. ax: 结果的低位
  2. dx: 结果的高16位
  3. cx: 余数

* description:

  * function:

    dword型数据(dx-ax, 32byte) / 除数(cx, 16byte) = 商(dx-ax, 32byte) + 余数(cx, 16byte)

  * method:

    * H = (X)h, L = (X)l
* int() 取整函数, rem() 取余函数
  
* X/N = int(H/N) * 2^16 + [rem(H/N) * 2^16 + L]/N
  
* code

  ```assembly
assume cs: code, ds: data
  
    data segment
          dw 4240h, 0fh, 0ah
    data ends
  
    code segment
          start:
        mov ax, data
          mov ds, ax
        mov ax, [0]
          mov dx, [2]
          mov cx, [4]
          call divdw
          divdw:
              ; init
              push si
              push bx
              ; 暂存ax(除数低位）
              mov si, ax
              ; 完成 H / N
              mov ax, dx
              mov dx, 0
              div cx
              ; 此时: dx - rem(H/N), ax - int(H/N)
              ; 暂存int(H/N)
              mov bx, ax
              mov ax, si
              div cx
              ; 此时ax - [rem(H/N) * 2^16 + L]/N
              mov cx, dx
              mov dx, bx
          return:
              pop bx
              pop si
              ret
      code ends
      
  end start
  
  
  ```

## dtoc

* patameter:

  * ax: word型数据
  * ds:si 指向字符串的首地址

* return

* description:

  将字符型数据(ax)以其所表示的十进制的形式在指定字符串首地址(ds:si)处写入

* code:

  ```assembly
  assume cs:code, ds:data
  	
      data segment
        db 10 dup (0)
      data ends
  
      code segment
      start:
      	; 初始化参数
          mov ax, data
          mov ds, ax
          mov si, 0
          mov ax, 12666
          call dtoc_init
  
      return:
          mov ax, 4c00h
          int 21h
      dtoc_init:
      	; 保护现场
          push dx
          push bx
          push cx
          ; 预先除数赋值
          mov bx, 10
      dtoc:
      	; word型数据高16位为0
          mov dx, 0
          div bx
          ; dl保存余数
          mov cx, dx 
          jcxz dtoc_return
          add dl, 30h
          mov ds:[si], dl
          inc si
          jmp dtoc
      dtoc_return:
          pop cx
          pop bx
          pop dx
          ret
      code ends
  
  end start
  ```
  
  ## ddtoc
  
  * patameter:
    * ax: dword型数据低位
    * dx: dword型数据高位
    * ds:si 指向字符串的首地址
  
  '4', '3', '2', '1', 0









