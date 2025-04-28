```python
import tkinter as tk
from tkinter import messagebox
import csv
from datetime import datetime

# Personel bilgilerini saklamak için bir liste
personel_listesi = []

# CSV dosyasına personel bilgilerini kaydetme fonksiyonu
def csv_kaydet():
    with open("personel_kayitlari.csv", mode="w", newline="", encoding="utf-8") as file:
        writer = csv.DictWriter(file, fieldnames=["TC", "AD", "SOYAD", "DOĞUM TARİHİ"])
        writer.writeheader()
        for personel in personel_listesi:
            writer.writerow(personel)

# Personel kaydı ekleme fonksiyonu
def personel_ekle():
    tc = entry_tc.get()
    ad = entry_ad.get().upper()  # AD alanını büyük harfe çevir
    soyad = entry_soyad.get().upper()  # SOYAD alanını büyük harfe çevir
    dogum_tarihi = entry_dogum_tarihi.get()

    # TC kimlik numarası 11 haneli mi kontrol et
    if len(tc) != 11 or not tc.isdigit():
        messagebox.showerror("Hata", "TC kimlik numarası 11 haneli olmalıdır ve sadece rakamlardan oluşmalıdır!")
        return  # Fonksiyondan çık

    # Aynı TC kimlik numarasıyla kayıt var mı kontrol et
    for personel in personel_listesi:
        if personel["TC"] == tc:
            messagebox.showerror("Hata", "Bu TC kimlik numarası zaten kayıtlı!")
            return

    # AD ve SOYAD sadece harflerden oluşmalı
    if not ad.isalpha():
        messagebox.showerror("Hata", "AD sadece harflerden oluşmalıdır!")
        return

    if not soyad.isalpha():
        messagebox.showerror("Hata", "SOYAD sadece harflerden oluşmalıdır!")
        return

    # Doğum tarihi formatını kontrol et (GG/AA/YYYY)
    try:
        datetime.strptime(dogum_tarihi, "%d/%m/%Y")
    except ValueError:
        messagebox.showerror("Hata", "Doğum tarihi 'GG/AA/YYYY' formatında olmalıdır!")
        return

    if tc and ad and soyad and dogum_tarihi:
        personel_listesi.append({
            "TC": tc,
            "AD": ad,
            "SOYAD": soyad,
            "DOĞUM TARİHİ": dogum_tarihi
        })
        messagebox.showinfo("Başarılı", "Personel başarıyla eklendi!")
        entry_tc.delete(0, tk.END)
        entry_ad.delete(0, tk.END)
        entry_soyad.delete(0, tk.END)
        entry_dogum_tarihi.delete(0, tk.END)
        # CSV dosyasını güncelle
        csv_kaydet()
    else:
        messagebox.showwarning("Hata", "Lütfen tüm alanları doldurun!")

# Personel listesini görüntüleme fonksiyonu
def personel_listele():
    listbox_personel.delete(0, tk.END)
    for personel in personel_listesi:
        listbox_personel.insert(tk.END, f"TC: {personel['TC']}, AD: {personel['AD']}, SOYAD: {personel['SOYAD']}, DOĞUM TARİHİ: {personel['DOĞUM TARİHİ']}")

# Personel silme fonksiyonu
def personel_sil():
    selected = listbox_personel.curselection()
    if not selected:
        messagebox.showwarning("Hata", "Lütfen silmek için bir personel seçin!")
        return
    index = selected[0]
    personel_listesi.pop(index)
    csv_kaydet()
    personel_listele()
    messagebox.showinfo("Başarılı", "Personel başarıyla silindi!")

# CSV dosyasından personel bilgilerini yükleme fonksiyonu
def csv_yukle():
    try:
        with open("personel_kayitlari.csv", mode="r", newline="", encoding="utf-8") as file:
            reader = csv.DictReader(file)
            for row in reader:
                personel_listesi.append(row)
    except FileNotFoundError:
        # Dosya yoksa yeni bir dosya oluşturulacak
        pass

# Ana pencereyi oluştur
root = tk.Tk()
root.title("Personel Kayıt Sistemi")

# Pencere boyutunu ayarla (800x600 piksel)
root.geometry("800x600")

# Pencere boyutunu yeniden boyutlandırılabilir yap
root.resizable(True, True)

# Satır ve sütunları yeniden boyutlandırılabilir yap
root.grid_rowconfigure(0, weight=1)
root.grid_rowconfigure(6, weight=1)
root.grid_columnconfigure(0, weight=1)
root.grid_columnconfigure(3, weight=1)

# Etiketler ve giriş alanları
label_tc = tk.Label(root, text="TC:")
label_tc.grid(row=1, column=1, padx=10, pady=5, sticky="e")
entry_tc = tk.Entry(root)
entry_tc.grid(row=1, column=2, padx=10, pady=5, sticky="w")

label_ad = tk.Label(root, text="AD:")
label_ad.grid(row=2, column=1, padx=10, pady=5, sticky="e")
entry_ad = tk.Entry(root)
entry_ad.grid(row=2, column=2, padx=10, pady=5, sticky="w")

label_soyad = tk.Label(root, text="SOYAD:")
label_soyad.grid(row=3, column=1, padx=10, pady=5, sticky="e")
entry_soyad = tk.Entry(root)
entry_soyad.grid(row=3, column=2, padx=10, pady=5, sticky="w")

label_dogum_tarihi = tk.Label(root, text="DOĞUM TARİHİ (GG/AA/YYYY):")
label_dogum_tarihi.grid(row=4, column=1, padx=10, pady=5, sticky="e")
entry_dogum_tarihi = tk.Entry(root)
entry_dogum_tarihi.grid(row=4, column=2, padx=10, pady=5, sticky="w")

# Butonlar
button_ekle = tk.Button(root, text="Personel Ekle", command=personel_ekle)
button_ekle.grid(row=5, column=1, columnspan=2, pady=10)

button_listele = tk.Button(root, text="Personel Listele", command=personel_listele)
button_listele.grid(row=6, column=1, columnspan=2, pady=10)

button_sil = tk.Button(root, text="Personel Sil", command=personel_sil)
button_sil.grid(row=7, column=1, columnspan=2, pady=10)

# Listbox ile personel listesi
listbox_personel = tk.Listbox(root, width=50)
listbox_personel.grid(row=8, column=1, columnspan=2, padx=10, pady=10)

# Çıkış butonu
button_exit = tk.Button(root, text="Çıkış", command=root.destroy)
button_exit.grid(row=9, column=1, columnspan=2, pady=10)

# CSV dosyasından verileri yükle
csv_yukle()

# Pencereyi başlat
root.mainloop()
```
