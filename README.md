# NASM 32-bit

- [Таблица системных вызовов для ввода-вывода](#таблица-системных-вызовов-для-ввода-вывода)
- [Программы](#программы)
  - [1. Циклический сдвиг массива](#1-циклически-сдвинуть-массив-1-2-3-на-1-позицию-влево-распечатать-результат-используя-прерывания)
  - [2. Побайтное сложение чисел](#2-сложить-побайтно-два-числа-числа--слова-использовать-только-регистры-размером-в-байт)
  - [3. Вычисление выражения z](#3-вычислить-значение-z-z10aab10bcd-a-b-c--байт-со-знаком-d---слово-со-знаком)
  - [4. Нахождение 10-го числа Фибоначчи](#4-найти-10-е-число-фибоначчи)
  - [5. Арифметическое выражение с LEA](#5-используя-команду-lea-реализуйте-арифметическое-выражение-c--10--a--b--14)
  - [6. Разделение массива на четные и нечетные элементы](#6-задан-массив-a-байт-со-знаком-сформировать-два-массива-включая-в-первый-массив-четные-элементы-исходного-массива-а-во-второй--нечетные)
  - [7. Поиск минимума и максимума в матрице](#L326)
  - [8. Проверка правильности скобок в выражении](#8-дана-строка--арифметическое-выражение-определить-правильность-расстановки-скобок-выдать-на-печать-yes-или-no)
  - [9. Рекурсивный факториал](#9-рекурсия-факториал-параметры-через-регистры)
  - [10. Рекурсивное вычисление числа Фибоначчи](#10-рекурсия--фибоначчи-параметры-через-регистры)
  - [11. Обратная польская нотация](#11-обратная-польская-нотация)
Документация [макро-библиотека ввода/вывода "st_io"](st_io.inc).

| Функция | Принимаемые/отдаваемые значения | Описание |
| --- | --- | --- |
| PRINT | `Символьная константа` | Вывод символьной константы в консоль |
| PUTCHAR | `'*', al, [c]` | Вывод символа в консоль |
| GETCHAR | `al` | Ввод символа из консоли |
| FINISH | | Стоп |
| UNSINT | `eax, [a]` | Вывод `unsigned int` в консоль |
| SIGNINT | `eax, [a]` | Вывод `signed int` в консоль |
| GETUN | `eax, [a]` | Ввод `unsigned int` из консоли |
| GETSIG | `eax, [a]` | Ввод `signed int` из консоли |

## Таблица системных вызовов для ввода-вывода

| Номер вызова (`eax`) | Название       | Описание                     | Регистры                                                                 |
|-----------------------|----------------|------------------------------|-------------------------------------------------------------------------|
| `3`                  | `sys_read`     | Чтение из файла/ввода        | `eax = 3`, `ebx = дескриптор файла` (0 для stdin), `ecx = адрес буфера`, `edx = длина буфера` |
| `4`                  | `sys_write`    | Запись в файл/вывод          | `eax = 4`, `ebx = дескриптор файла` (1 для stdout), `ecx = адрес строки`, `edx = длина строки` |
| `1`                  | `sys_exit`     | Завершение программы         | `eax = 1`, `ebx = код возврата` (обычно 0)                              |

>[!NOTE]
>После `sys_read` в `eax` возвращается количество прочитанных байт.

## Программы

### 1. Циклически сдвинуть массив [1, 2, 3] на 1 позицию влево. Распечатать результат, используя прерывания

```asm
section .data
    A db '1', '2', '3', 0   ; Массив символов и нулевой терминатор
    len equ $ - A - 1       ; Длина массива (3 элемента)

section .text
global _start
_start:
    ; Циклический сдвиг массива влево на 1 позицию
    mov al, [A]             ; Сохраняем первый элемент (будет перемещен в конец)
    mov ecx, len - 1        ; Счетчик для сдвига (len-1 итераций)
    mov esi, 1              ; Индекс для доступа к элементам, начиная с 1
shift_loop:
    mov bl, [A + esi]       ; Берем следующий элемент
    mov [A + esi - 1], bl   ; Сдвигаем его влево
    inc esi                 ; Переходим к следующему элементу
    loop shift_loop         ; Повторяем, пока ecx > 0
    mov [A + len - 1], al   ; Помещаем первый элемент в конец

    ; Вывод массива
    mov ecx, len            ; Счетчик для вывода (кол-во элементов)
    mov esi, 0              ; Индекс для начала массива
print_loop:
    push ecx                ; Сохраняем счетчик
    mov eax, 4              ; sys_write
    mov ebx, 1              ; stdout
    lea ecx, [A + esi]      ; Адрес текущего элемента
    mov edx, 1              ; Длина вывода (1 символ)
    int 0x80                ; Вызов ядра для вывода
    inc esi                 ; Следующий элемент
    pop ecx                 ; Восстанавливаем счетчик
    loop print_loop         ; Повторяем для всех элементов

    ; Завершение программы
    mov eax, 1              ; sys_exit
    xor ebx, ebx            ; Код возврата 0
    int 0x80                ; Вызов ядра 
```

### 2. Сложить побайтно два числа (Числа – слова, использовать только регистры размером в байт)

```asm
%include "st_io.inc"

section .data
    A dw 254
    B dw 250

section .text
global _start
_start:
    mov al, [A]
    mov bl, [B]
    add al, bl; Add A[0] + B[0]
    mov [A], al  

    mov ah, [A + 1]
    mov bh, [B + 1]
    add ah, bh              ; Add A[1] + B[1]
    mov [A + 1], ah

    movzx eax, word [A]
    SIGNINT eax
    PUTCHAR 10

    FINISH
```

### 3. Вычислить значение z: z=10*a/(a+b)+10*b/(c+d); a, b, c – байт со знаком, d - слово со знаком

```asm
%include "st_io.inc"

section .data
    a db 12  
    b db 107   
    c db -33  
    d dw -27   

section .text
    global _start

_start:
    mov al, [a] 
    mov bl, [b] 
    add al, bl  
    movsx bx, al

    mov al, [c]      
    movsx cx, al     
    mov ax, [d]      
    add cx, ax       

    mov al, 10   
    imul byte [a]

    cwd       
    idiv bx   
    mov bx, ax

    mov al, 10   
    imul byte [b]

    cwd    
    idiv cx

    add bx, ax

    movsx ebx, bx  ; Fix: Sign-extend bx to ebx fully

    SIGNINT ebx
    PUTCHAR 10

    mov ax, 1
    xor bx, bx
    int 0x80
```

### 4. Найти 10-е число Фибоначчи

```asm
%include "st_io.inc"

section .data
    tmp dd 0
    result dd 0

section .text
    global _start

_start:
    xor cx, cx
    mov cx, 1
    mov eax, 1
    mov ebx, 1
    xor edx, edx

Cycle:
    mov [tmp], eax
    add eax, ebx
    mov ebx, [tmp]
    mov edx, eax
    inc cx
    cmp cx, 8
    je Ended
    jmp Cycle
Ended:
    mov [result], edx

    SIGNINT [result]
    PUTCHAR 10
    FINISH
```

### 5. Используя команду LEA, реализуйте арифметическое выражение: c = 10 * a + b + 14

```asm
%include "st_io.inc"
; c = 10 * a + b + 14 
section .data
    a dd -5         
    b dd -13         
    c dd 0          

section .text
    global _start

_start:
    
    mov eax, [a]        
    lea ebx, [eax * 8]  
    lea ecx, [eax * 2]  
    add ebx, ecx        
    add ebx, [b]        
    lea ecx, [ebx + 14] 
    mov [c], ecx        

    SIGNINT ecx
    
    mov eax, 1          
    xor ebx, ebx        
    int 0x80            
```

### 6. Задан массив a (байт со знаком). Сформировать два массива, включая в первый массив четные элементы исходного массива, а во второй – нечетные

```asm
%include "st_io.inc"

section .data
    
    a db 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 115
    a_len equ $ - a

    
    even_array db 20 dup(0)
    odd_array  db 20 dup(0)

    
    even_len dd 0
    odd_len  dd 0

section .bss
    
    num_buffer resb 12

section .text
global _start

_start:
    
    xor ecx, ecx          
    xor esi, esi          
    xor edi, edi          

process_array:
    
    cmp ecx, a_len
    jge print_arrays      

    movsx eax, byte [a + ecx]  

    test al, 1
    jnz odd_element       

    mov [even_array + esi], al
    inc esi               
    inc dword [even_len]  
    jmp next_element

odd_element:
    
    mov [odd_array + edi], al
    inc edi               
    inc dword [odd_len]   

next_element:
    
    inc ecx
    jmp process_array

print_arrays:
    PRINT "Четные элементы: "

    xor ecx, ecx          
print_even:
    cmp ecx, [even_len]
    jge print_odd         

    movsx eax, byte [even_array + ecx]
    SIGNINT eax
    PUTCHAR ' '

    inc ecx
    jmp print_even

print_odd:
    PUTCHAR 10
    PRINT "Нечетные элементы: "

    xor ecx, ecx          
print_odd_loop:
    cmp ecx, [odd_len]
    jge exit              

    movsx eax, byte [odd_array + ecx]
    SIGNINT eax
    PUTCHAR ' '

    inc ecx
    jmp print_odd_loop

exit:
    PUTCHAR 10
    
    mov eax, 1            
    xor ebx, ebx          
    int 0x80
```

### 7. Найти минимальный и максимальный элементы матрицы и вывести их значения, номер строки и столбца, в которых они находятся

```asm
%include "st_io.inc"

section .data
    matrix dd 1, 204, -3, 4, 32, 75  
    cols    dd 2                    
    rows    dd 3                    
    max_elem dd -2147483640       
    max_row_index dd 0              
    max_col_index dd 0              
    min_elem dd 2147483640          
    min_row_index dd 0              
    min_col_index dd 0              

section .text
    global _start

_start:
    xor esi, esi
    mov ecx, [rows]            

row_loop:
    mov ebp, ecx            
    xor edi, edi        
    mov ecx, [cols]  

col_loop:
    
    mov eax, esi                    
    mov ebx, [cols]                 
    shl ebx, 2                      
    mul ebx                         
    mov ebx, edi                    
    shl ebx, 2                      
    add eax, ebx                    

    
    mov ebx, [matrix + eax]

    
    next:
    cmp ebx, [max_elem]
    jg update_max_elem              

    
    cmp ebx, [min_elem]
    jl update_min_elem              

    inc edi              ; j++
    loop col_loop

    mov ecx, ebp
    inc esi              ; i++
    loop row_loop                   

    
    PRINT "Max_element: "
    SIGNINT [max_elem]
    PUTCHAR 10
    PRINT "i: "
    SIGNINT [max_row_index]
    PUTCHAR 10
    PRINT "j: "
    SIGNINT [max_col_index]
    PUTCHAR 10
    PUTCHAR 10
    PRINT "Min_element: "
    SIGNINT [min_elem]
    PUTCHAR 10
    PRINT "i: "
    SIGNINT [min_row_index]
    PUTCHAR 10
    PRINT "j: "
    SIGNINT [min_col_index]
    PUTCHAR 10

    FINISH

update_max_elem:
    mov [max_elem], ebx             
    mov [max_row_index], esi        
    mov [max_col_index], edi        
    jmp next                        

update_min_elem:
    mov [min_elem], ebx             
    mov [min_row_index], esi        
    mov [min_col_index], edi        
    jmp next
```

### 8. Дана строка – арифметическое выражение. Определить правильность расстановки скобок. Выдать на печать YES или NO

```asm
%include "st_io.inc"

section .data
    expr db "(2 + 3) * (5) - (1 + 2))", 0   
    ;expr db "2 + 3) * (5 - 1", 0         

section .text
    global _start

_start:
    mov ebx, esp        

check_loop:
    movzx eax, byte [expr + ecx]  
    cmp eax, 0          
    je check_result

    cmp eax, '('        
    je open_bracket
    cmp eax, ')'        
    je close_bracket

    inc ecx             
    jmp check_loop

open_bracket:
    push 1              
    inc ecx
    jmp check_loop

close_bracket:
    cmp esp, ebx        
    je error            
    pop eax             
    inc ecx
    jmp check_loop

check_result:
    cmp esp, ebx        
    je success          

error:
    PRINT "NO"
    jmp exit

success:
    PRINT "YES"

exit:
    PUTCHAR 10
    mov eax, 1
    mov ebx, 0
    int 0x80
```

### 9. Рекурсия факториал (параметры через регистры)

```asm
%include "st_io.inc"
section .text
global _start

_start:
    
    mov eax, 5
    call factorial

    SIGNINT eax
    
    FINISH

factorial:
    push ebp
    mov ebp, esp
    push ebx

    cmp eax, 1
    jle .label1
    
    mov ebx, eax
    dec eax
    call factorial
    mul ebx

    jmp .exit

.label1:
    mov eax, 1

.exit:
    pop ebx
    mov esp, ebp
    pop ebp
    ret
```

### 10. Рекурсия – Фибоначчи (параметры через регистры)

```asm
%include "st_io.inc"
section .text
global _start

_start:
    mov eax, 4
    call fibonacci
    SIGNINT eax
    PRINT 10
    FINISH

fibonacci:
    push ebp
    mov ebp, esp 
    push ebx
    
    cmp eax, 0
    je .zero
    cmp eax, 1
    je .one

    push eax
    dec eax 
    call fibonacci

    mov ebx, eax
    pop eax
    sub eax, 2

    call fibonacci
    add eax, ebx 
    jmp .exit

.zero:
    mov eax, 0
    jmp .exit

.one:
    mov eax, 1

.exit:
    pop ebx 
    mov esp, ebp 
    pop ebp 
    ret
```

### 11. Обратная польская нотация

```asm
section .data
    ; Пример выражения в ОПН: "3 4 + 2 *"
    expr db "3 4 + 2 *", 0
    expr_len equ $ - expr - 1

section .text
global _start

_start:
    xor esi, esi          ; Индекс в строке выражения

parse_loop:
    cmp esi, expr_len     ; Проверяем конец строки
    jge evaluate_done

    mov al, [expr + esi]  ; Читаем текущий символ
    cmp al, ' '           ; Пропускаем пробелы
    je next_char

    ; Проверяем, является ли символ числом (0-9)
    cmp al, '0'
    jl check_operator
    cmp al, '9'
    jg check_operator

    ; Если число, пушим его в стек
    sub al, '0'           ; Преобразуем символ в число
    movzx eax, al         ; Расширяем до 32 бит
    push eax              ; Помещаем число в стек
    jmp next_char

check_operator:
    ; Проверяем, является ли символ оператором
    cmp al, '+'
    je handle_operator
    cmp al, '-'
    je handle_operator
    cmp al, '*'
    je handle_operator
    cmp al, '/'
    je handle_operator
    jmp next_char         ; Если не число и не оператор, пропускаем

handle_operator:
    ; Извлекаем два операнда из стека
    pop ebx               ; Второй операнд (ebx)
    pop eax               ; Первый операнд (eax)

    ; Выполняем операцию в зависимости от символа
    cmp byte [expr + esi], '+'
    je do_add
    cmp byte [expr + esi], '-'
    je do_sub
    cmp byte [expr + esi], '*'
    je do_mul
    cmp byte [expr + esi], '/'
    je do_div

do_add:
    add eax, ebx          ; eax = eax + ebx
    jmp push_result
do_sub:
    sub eax, ebx          ; eax = eax - ebx
    jmp push_result
do_mul:
    imul eax, ebx         ; eax = eax * ebx
    jmp push_result
do_div:
    cdq                   ; Расширяем eax до edx:eax для деления
    idiv ebx              ; eax = eax / ebx
    jmp push_result

push_result:
    push eax              ; Помещаем результат обратно в стек

next_char:
    inc esi               ; Переходим к следующему символу
    jmp parse_loop

evaluate_done:
    ; Результат остается в стеке на вершине
    ; Завершение программы
    mov eax, 1            ; sys_exit
    xor ebx, ebx          ; Код возврата 0
    int 0x80
```