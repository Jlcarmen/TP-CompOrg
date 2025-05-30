# TP-CompOrg
---

# Processor-Control-Instructions

## Decimal to 32-bit Binary Converter (Linux Assembly - NASM)

This is a simple assembly program I made that takes a signed decimal number from the user and converts it to a 32-bit binary format. It runs on Linux and works well in online assemblers like [JDoodle](https://www.jdoodle.com/).

---

## Program Structure

The program is split into three main sections:

* **.data** — Where I store the messages (like prompts and output texts).
* **.bss** — This is where I reserve space for user input and the binary output.
* **.text** — Contains the actual instructions, starting from `_start`.

---

## How It Works

### 1. **Input**

* It first asks the user to enter a signed decimal number.
* It reads the input using `sys_read`.
* I made sure it removes the newline character and replaces it with a null terminator (`\0`) so it doesn't mess up the conversion.

### 2. **Decimal to Integer**

* The program handles both positive and negative numbers.
* It checks for a '+' or '-' sign at the start.
* Then it loops through each character, converts it from ASCII to a number, and builds the full integer in `EBX`.
* If the number is negative, it flips it using `NEG`.

### 3. **Binary Conversion**

* It converts the number to binary by checking each bit (starting from the least significant one).
* Each bit is converted to a '0' or '1' character.
* These characters are stored in reverse (from right to left) to build the final binary string.
* The result is printed using `sys_write`.

### 4. **Looping**

* After printing the binary version, the program loops back and waits for another input.
* If the user enters `0`, it exits using `sys_exit`.

---

## What I Learned

This project helped me understand:

* Linux system calls (`read`, `write`, and `exit`)
* ASCII and number conversions
* Bitwise operations
* How to handle strings and loops in assembly

---

## How to Run

You can run it using JDoodle or any online Linux NASM assembler.

Just:

1. Paste the code,
2. Assemble and run,
3. Type a number and see its binary version printed,
4. Enter `0` to exit the program.

---

```nasm
section .data
    prompt_msg db 'Enter a decimal: (0 to exit) ', 0
    prompt_len equ $ - prompt_msg
    newline    db 10, '', 0
    binary_msg db 'Binary form: ', 0

section .bss
    input_buffer resb 12
    binary_output resb 33

section .text
    global _start

_start:
main_loop:
    ; Display prompt
    mov eax, 4
    mov ebx, 1
    mov ecx, prompt_msg
    mov edx, prompt_len
    int 0x80

    ; Read input
    mov eax, 3
    mov ebx, 0
    mov ecx, input_buffer
    mov edx, 11
    int 0x80

    ; Handle newline
    mov ecx, eax
    mov edi, input_buffer
check_bytes:
    cmp ecx, 0
    je null_terminate_end
    cmp byte [edi], 10
    je terminate_at_newline
    inc edi
    dec ecx
    jmp check_bytes

terminate_at_newline:
    mov byte [edi], 0
    jmp check_exit

null_terminate_end:
    cmp eax, 0
    je check_exit
    mov byte [input_buffer + eax - 1], 0

check_exit:
    cmp byte [input_buffer], '0'
    je exit_program

    ; Decimal to integer
    mov esi, input_buffer
    mov ebx, 0
    mov ecx, 0

    cmp byte [esi], '-'
    jne check_plus_convert
    inc ecx
    inc esi
    jmp convert_digit

check_plus_convert:
    cmp byte [esi], '+'
    jne convert_digit
    inc esi

convert_digit:
    movzx eax, byte [esi]
    cmp al, '0'
    jl conversion_done
    cmp al, '9'
    jg conversion_done
    sub al, '0'

    push eax
    mov eax, ebx
    mov edx, 10
    mul edx
    mov ebx, eax
    pop eax
    add ebx, eax

    inc esi
    jmp convert_digit

conversion_done:
    cmp ecx, 1
    jne convert_to_binary
    neg ebx

convert_to_binary:
    mov edi, binary_output + 32
    mov byte [edi], 0

    mov ecx, 32
binary_conversion_loop:
    dec edi
    mov eax, ebx
    and eax, 1
    add al, '0'
    mov [edi], al
    shr ebx, 1
    loop binary_conversion_loop

    ; Display binary output
    mov eax, 4
    mov ebx, 1
    mov ecx, binary_msg
    mov edx, 13
    int 0x80

    mov eax, 4
    mov ebx, 1
    mov ecx, binary_output
    mov edx, 32
    int 0x80

    ; Newline
    mov eax, 4
    mov ebx, 1
    mov ecx, newline
    mov edx, 1
    int 0x80

    jmp main_loop

exit_program:
    mov eax, 1
    xor ebx, ebx
    int 0x80
```

---



