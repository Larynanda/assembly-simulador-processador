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

    ; Registradores 128-bit
    R1280 times 16 db 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16
    R1281 times 16 db 16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1

    ; Flags
    ZF db 0
    CF db 0
    NF db 0

    fmt8 db "R8[%d] = %u", 10, 0
    fmt16 db "R16[%d] = %u", 10, 0
    fmt32 db "R32[%d] = %u", 10, 0
    fmt128 db "R128[%d] = %02X %02X %02X %02X %02X %02X %02X %02X %02X %02X %02X %02X %02X %02X %02X %02X", 10, 0
    fmt_msg db "Execução finalizada.", 10, 0

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
.copy_loop:
    mov al, [rsi]
    mov [rdi], al
    inc rsi
    inc rdi
    loop .copy_loop

    call update_flags_128

    ;Impressão dos registradores
    call print_regs_8
    call print_regs_16
    call print_regs_32
    call print_regs_128

    mov rdi, fmt_msg
    xor eax, eax
    call printf

    mov rax, 60         
    xor rdi, rdi        
    syscall

add_128:
    xor r8, r8          
    mov rcx, 16
.loop:
    mov al, [rsi]
    mov bl, [rdi]
    add al, bl
    add al, r8b         
    mov [rdx], al

    mov r8b, 0
    cmp al, bl
    jb .set_carry
    cmp al, 0
    jne .no_carry
.set_carry:
    mov r8b, 1
.no_carry:

    inc rsi
    inc rdi
    inc rdx
    loop .loop
    ret

update_flags_128:
    xor al, al
    mov rcx, 16
    mov rsi, R1280
.check_zero:
    or al, [rsi]
    inc rsi
    loop .check_zero

    cmp al, 0
    sete byte [ZF]

    mov al, [R1280 + 15]
    shr al, 7
    mov [NF], al

    mov byte [CF], 0
    ret

; Imprimir registradores de 8 bits
print_regs_8:
    mov esi, 0
    mov edi, fmt8
    movzx edx, byte [R80]
    xor eax, eax
    call printf

    inc esi
    mov edi, fmt8
    movzx edx, byte [R81]
    xor eax, eax
    call printf

    inc esi
    mov edi, fmt8
    movzx edx, byte [R82]
    xor eax, eax
    call printf

    inc esi
    mov edi, fmt8
    movzx edx, byte [R83]
    xor eax, eax
    call printf
    ret


; Imprimir registradores de 16 bits
print_regs_16:
    mov esi, 0
    mov edi, fmt16
    movzx edx, word [R160]
    xor eax, eax
    call printf

    inc esi
    mov edi, fmt16
    movzx edx, word [R161]
    xor eax, eax
    call printf

    inc esi
    mov edi, fmt16
    movzx edx, word [R162]
    xor eax, eax
    call printf

    inc esi
    mov edi, fmt16
    movzx edx, word [R163]
    xor eax, eax
    call printf
    ret


; Imprimir registradores de 32 bits
print_regs_32:
    mov esi, 0
    mov edi, fmt32
    mov edx, [R320]
    xor eax, eax
    call printf

    inc esi
    mov edi, fmt32
    mov edx, [R321]
    xor eax, eax
    call printf

    inc esi
    mov edi, fmt32
    mov edx, [R322]
    xor eax, eax
    call printf

    inc esi
    mov edi, fmt32
    mov edx, [R323]
    xor eax, eax
    call printf
    ret


; Imprimir registrador de 128 bits (R1280)
print_regs_128:
    mov edi, fmt128
    mov esi, 0    

    mov rdx, [R1280]
    mov rcx, [R1280+2]
    mov r8,  [R1280+4]
    mov r9,  [R1280+6]

    xor eax, eax
    mov rdx, [R1280]           
    mov rcx, [R1280+8]         

    mov r8b,  [R1280]
    mov r9b,  [R1280+1]
    mov r10b, [R1280+2]
    mov r11b, [R1280+3]
    mov r12b, [R1280+4]
    mov r13b, [R1280+5]
    mov r14b, [R1280+6]
    mov r15b, [R1280+7]

    push r15
    push r14
    push r13
    push r12
    push r11
    push r10
    push r9
    push r8

    mov r8b,  [R1280+8]
    mov r9b,  [R1280+9]
    mov r10b, [R1280+10]
    mov r11b, [R1280+11]
    mov r12b, [R1280+12]
    mov r13b, [R1280+13]
    mov r14b, [R1280+14]
    mov r15b, [R1280+15]

    xor eax, eax
    call printf

    pop r8
    pop r9
    pop r10
    pop r11
    pop r12
    pop r13
    pop r14
    pop r15

    ret
