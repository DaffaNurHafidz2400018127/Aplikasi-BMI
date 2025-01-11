# Aplikasi-BMI

.model small
.stack 100h
.data   
    prompt db 'Masukkan berat badan (kg): $'
    prompt2 db 0Ah, 0Dh, 'Masukkan tinggi badan (cm): $'
    hasil db 0Ah, 0Dh, 'BMI Anda adalah: $', 0Ah, 0Dh, '$'
    kategori db 0Ah, 0Dh, 'Kategori BMI: $', 0Ah, 0Dh, '$'
    newline db 0Ah, 0Dh, '$'
    error_msg db 0Ah, 0Dh, 'Input tidak valid!$'

    ; Variabel untuk menyimpan input dan hasil
    berat dw 0   ; Menggunakan 2 byte untuk menyimpan nilai berat badan (integer)
    tinggi dw 0  ; Menggunakan 2 byte untuk menyimpan tinggi badan (integer)
    bmi dw 0     ; Variabel untuk menyimpan hasil BMI

.code

main:
    ; Inisialisasi segment data
    mov ax, @data
    mov ds, ax

    ; Menampilkan prompt untuk memasukkan berat badan
    lea dx, prompt
    mov ah, 09h
    int 21h

    ; Input berat badan
    call get_input
    mov [berat], ax    ; Menyimpan berat badan ke dalam variabel berat

    ; Menampilkan prompt untuk memasukkan tinggi badan dalam cm
    lea dx, prompt2
    mov ah, 09h
    int 21h

    ; Input tinggi badan
    call get_input
    mov [tinggi], ax   ; Menyimpan tinggi badan ke dalam variabel tinggi

    ; Menghitung tinggi badan dalam meter (tinggi / 100)
    mov ax, [tinggi]   ; Salin tinggi badan dalam cm ke register ax
    mov bx, 100        ; Menyimpan angka 100 ke register bx
    xor dx, dx         ; Bersihkan dx sebelum pembagian
    div bx             ; Membagi ax dengan 100, hasilnya dalam ax (meter)

    ; Menghitung tinggi * tinggi (dalam meter)
    mov bx, ax         ; Salin hasil tinggi dalam meter ke bx
    mov ax, [berat]    ; Salin berat badan ke dalam ax
    mov cx, ax         ; Simpan berat badan untuk pembagian nanti
    mul bx             ; ax = tinggi * tinggi (meter^2)

    ; Pastikan hasil tinggi * tinggi tidak 0
    cmp ax, 0
    je invalid_input   ; Jika ax = 0, tampilkan pesan error

    ; Sekarang ax berisi tinggi * tinggi (m^2), bagi dengan berat badan
    mov ax, cx         ; Pindahkan berat badan ke ax
    xor dx, dx         ; Clear dx untuk hasil pembagian
    div bx             ; Membagi ax dengan hasil tinggi * tinggi

    ; Simpan hasil BMI
    mov [bmi], ax

    ; Menampilkan hasil BMI
    lea dx, hasil
    mov ah, 09h
    int 21h

    ; Menampilkan kategori BMI
    lea dx, kategori
    mov ah, 09h
    int 21h

    ; Tentukan kategori BMI berdasarkan hasil
    mov ax, [bmi]
    cmp ax, 1850        ; BMI < 18.5 -> Kurus
    jb kurang
    cmp ax, 2500        ; 18.5 <= BMI < 24.9 -> Normal
    ja gemuk
    jmp normal

kurang:
    lea dx, kurang_msg
    mov ah, 09h
    int 21h
    jmp end_program

normal:
    lea dx, normal_msg
    mov ah, 09h
    int 21h
    jmp end_program

gemuk:
    lea dx, gemuk_msg
    mov ah, 09h
    int 21h
    jmp end_program

invalid_input:
    ; Menampilkan pesan error jika input tidak valid
    lea dx, error_msg
    mov ah, 09h
    int 21h

end_program:
    ; Menampilkan newline setelah output
    lea dx, newline
    mov ah, 09h
    int 21h
    mov ah, 4Ch
    int 21h

get_input proc
    xor ax, ax          ; Clear ax
    xor cx, cx          ; Clear cx
input_loop:
    mov ah, 01h         ; Input karakter
    int 21h
    cmp al, 0Dh         ; Jika Enter ditekan
    je done_input
    sub al, '0'         ; Convert ASCII ke angka
    mov dx, ax
    shl ax, 1           ; ax = ax * 2
    shl ax, 1           ; ax = ax * 4
    add ax, dx          ; ax = ax * 10
    add ax, dx          ; ax = ax * 10 + digit
    jmp input_loop
done_input:
    ret
get_input endp

; Pesan kategori BMI
kurang_msg db 'Kurus$', 0
normal_msg db 'Normal$', 0
gemuk_msg db 'Gemuk$', 0

end main
