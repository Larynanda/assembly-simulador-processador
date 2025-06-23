# assembly-simulador-processador
Simulador de registradores em Assembly com operações 8, 16, 32 e 128 bits.
section .data
    ; Registradores 8-bit
    R80 db 10
    R81 db 5
    R82 db 0
    R83 db 0

    ; Registradores 16-bit
    R160 dw 1000
    R161 dw 500
    R162 dw 0
    R163 dw 0

    ; Registradores 32-bit
    R320 dd 10000
    R321 dd 5000
    R322 dd 0
    R323 dd 0

    ; Registradores 128-bit (16 bytes cada)
    R1280 times 16 db 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16
    R1281 times 16 db 16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1

    ; Flags
    ZF db 0
    CF db 0
    NF db 0

    DESV dd 0

    fmt8 db "R8[%d] = %u",10,0
    fmt16 db "R16[%d] = %u",10,0
    fmt32 db "R32[%d] = %u",10,0
    fmt128 db "R128[%d] = %016X%016X%016X%016X",10,0

    fmt_msg db "Execução finalizada.",10,0

section .bss
    tmp128 resb 16

section .text
global _start
extern printf

_start:
    mov rsi, R1280
    mov rdi, R1281
    mov rdx, tmp128
    call add_128

    mov rcx, 16
    mov rsi, tmp128
    mov rdi, R1280
.copy_128:
    mov al, [rsi]
    mov [rdi], al
    inc rsi
    inc rdi
    loop .copy_128

    mov rsi, R1280
    call update_flags_128

    ; Imprimir registradores 8 bits
    mov edi, fmt8
    mov esi, 0
    movzx eax, byte [R80]
    mov edx, eax
    xor eax, eax
    call printf

    mov edi, fmt8
    mov esi, 1
    movzx eax, byte [R81]
    mov edx, eax
    xor eax, eax
    call printf

    mov edi, fmt8
    mov esi, 2
    movzx eax, byte [R82]
    mov edx, eax
    xor eax, eax
    call printf

    mov edi, fmt8
    mov esi, 3
    movzx eax, byte [R83]
    mov edx, eax
    xor eax, eax
    call printf

    ; Imprimir registradores 16 bits
    mov edi, fmt16
    mov esi, 0
    mov ax, [R160]
    movzx edx, ax
    xor eax, eax
    call printf

    mov edi, fmt16
    mov esi, 1
    mov ax, [R161]
    movzx edx, ax
    xor eax, eax
    call printf

    mov edi, fmt16
    mov esi, 2
    mov ax, [R162]
    movzx edx, ax
    xor eax, eax
    call printf

    mov edi, fmt16
    mov esi, 3
    mov ax, [R163]
    movzx edx, ax
    xor eax, eax
    call printf

    ; Imprimir registradores 32 bits
    mov edi, fmt32
    mov esi, 0
    mov edx, [R320]
    xor eax, eax
    call printf

    mov edi, fmt32
    mov esi, 1
    mov edx, [R321]
    xor eax, eax
    call printf

    mov edi, fmt32
    mov esi, 2
    mov edx, [R322]
    xor eax, eax
    call printf

    mov edi, fmt32
    mov esi, 3
    mov edx, [R323]
    xor eax, eax
    call printf

    ; Imprimir registradores 128 bits
    mov edi, fmt128
    mov esi, 0           
    mov rax, [R1280 + 0]
    mov rbx, [R1280 + 8]
    mov rcx, [R1280 + 16] 
    mov rdx, [R1280 + 24]

    mov rax, [R1280]
    mov rbx, [R1280+8]
    mov rdx, rbx
    mov rcx, rax

    mov edi, fmt128
    mov esi, 0
    mov rsi, 0           
    mov rdx, [R1280]
    mov rcx, [R1280+8]
    mov r8, 0          
    mov r9, 0

    xor eax, eax
    call printf

    mov rdi, fmt_msg
    xor eax, eax
    call printf

    mov rax, 60
    xor rdi, rdi
    syscall

add_128:
    xor r8, r8        ; carry = 0
    mov rcx, 16
.add_loop:
    mov al, [rsi]
    mov bl, [rdi]
    mov ah, r8b       ; carry
    add al, bl
    add al, ah
    mov [rdx], al

    mov r8b, 0
    cmp al, bl
    jb .carry_set
    cmp al, 0
    jne .carry_clear
.carry_set:
    mov r8b, 1
.carry_clear:

    inc rsi
    inc rdi
    inc rdx
    loop .add_loop
    ret

update_flags_128:
    mov rcx, 16
    mov rsi, R1280
    xor al, al

.check_zero:
    mov dl, [rsi]
    or al, dl
    inc rsi
    loop .check_zero

    cmp al, 0
    sete byte [ZF]

    mov al, [R1280+15]
    shr al, 7
    mov [NF], al

    ret
