## Instaliranje ArchLinux-a, zajedno sa Windows-om u UEFI-modu
## Ubacimo Arch na flash ili na disk
## Prilikom butovanja pritiskamo taster na tastaturi da bismo usli u boot meni
## Bitno je da odaberemo opciju za butovanja sa USB-a i to u UEFI-mod
## Zatim treba da izlistamo sve particije koje se nalaze na hard-disku komandom

fdisk -l

## Izlaz komande treba da izgleda ovako:

## Disk /dev/sda: 931.5 GiB, 1000204886016 bytes, 1953525168 sectors
## Units: sectors of 1 * 512 = 512 bytes
## Sector size (logical/physical): 512 bytes / 4096 bytes
## I/O size (minimum/optimal): 4096 bytes / 4096 bytes
## Disklabel type: gpt
## Disk identifier: 18A2918F-A3B3-4D83-8197-6A773641755B

## Device          Start        End    Sectors   Size Type
## /dev/sda1        2048     923647     921600   450M Windows recovery environment
## /dev/sda2      923648    1126399     202752    99M EFI System
## /dev/sda3     1126400    1159167      32768    16M Microsoft reserved
## /dev/sda4     1159168 1134323711 1133164544 540.3G Microsoft basic data
## Free Space                                  400.0G 

## Potrebno je sada da obrisemo prethodnu installaciju Linux-a iz boot-managera, ako je Linux prethodno instaliran, ako nije ovaj korak preskacemo

## Ako je Linux prethodno instaliran onda: 

efibootmgr

## Zatim:

efibootmgr -b 0 -B 

## 0 je broj u boot sektoru na kojem se nalazi Linux a to se vidi kao izlaz iz efibootmgr
## Ovu komdanu je potrebno pokrenuti 2 puta ako smo prilikom prethodne instalacije sami menjali izlgled boot-managera

efibootmgr -b 3 -B 

## Linux je bio na tom mestu prilikom prvobitne instalacije

## Sada prelazimo na particionisanje diska. Preporucljivo je da se razdvoje /root i /home particija kao i swap

## Za particionisanje GPT particija koriste se 2 alata gdisk, ili cgdisk koji nudi graficki interfejs

gdisk /dev/sdX ## X je ime hard-disk i varira

## neke komdane u gdisk-u, koje su nam korisne: 
## p stampa tabelu sa particijama
## d brise particiju koju smo odabrali 
## n pravi novu particiju GPT-tipa
## t menja tip odredjene particije
## w upisuje sve izmene koje smo do tada napravili na disk

## Ako je na racunaru vec postojao Linux, onda je potrebno da izbrisemo /root i swap komdanom "d"
## Da bismo znali na kojim particijama se nalaze /root i swap u gdisku unosimo komadnu "p"
## Ovo radimo da bismo napravili slobodan prostor na kom cemo kasnije instalirati Linux
## Sada kada smo izbrisali prethodnu instalaciju Linuxa sa diska potrebno je da napravimo nove particije

## U gdisk-u, to je moguce preko komande "n"
## Prvo treba da napravimo swap
## gdisk ce nas pitati da unesemo velicinu prvog sektora (First sector), ovde ne treba nista menjati pritiskamo Enter
## Last sector, tojest velicina poslednjeg sektora je ona koju treba da menjamo, to je ustvari velicina swapa
## Kada nas gdisk pita da unesemo velicinu poslednjeg sektora treba da upisemo "+" i velicinu koju zelimo
## Velicina swap-a treba da bude {swap(size)=1.5*RAM}, u slucaju da nam je ram 4GiB pisemo "+6G"
## Onda ce nas gdisk pitati da unesemo HEX code, za SWAP to je "8200"
## Tako smo napravili SWAP
## Sada treba da napravimo /root i /home
## Prvo pravimo /home komdanom "n"
## First sector,treba da ostavimo kako jeste, u Last sector-u unosimo velicinu /home-a
## Hex code je 8300 dakle samo pritisnemo Enter
## Za pravljennje /root sa "n" pravimo, First i Last sektor treba ostaviti kako je ponudjeno dakle pritiskamo Enter
## Hex code isto Enter
## Sada treba proveriti kako izgleda particiona tabela, tojest da vidimo sta smo uradili
## Komdanda p stampa tabelu svih particija na disku
## Ako smo zadovoljni tabelom, treba da upisemo na disk sve sto smo uradili i to preko komade "w"
## Particije su napravljene ali treba im dodeliti FileSystem tip
## Komanda mkfs upravo to radimo

mkfs.ext4 -L HOME /dev/sdX

## Ovom komandom formatiramo HOME particiju na tip EXT4 /dev/sdX je naziv particije na kojoj se nalazi HOME

mkfs.ext4 -L ARCH /dev/sdX 

## Ovom komandom formatiramo ROOT particiju na tip EXT4 /dev/sdX je naziv particije na kojoj se nalazi ROOT

mkswap -L SWAP /dev/sdX 

## Ovom komandom formatiramo SWAP particiju na SWAP tip
## Sada je potrebno da ukljucimo SWAP

swapon /dev/sdX 

## /dev/sdX je naziv particije na kojoj se nalazi SWAP
## Sada kada smo formatirali particije potrebno je da ih vezemo sa pravi /root i /home

mont /dev/sdX /mnt 

## Ovako smo napravili Root
## /dev/sdX je ime Root particije koju smo prethono napravili 

mkdir /mnt/boot
mount /dev/sdX /mnt/boot

## S obzirom da je Windows prilikom instalacije vec napravio EFI boot particije koju ce racunar cita kada se bootuje
## Treba da je vezemo za boot direktorijum na /root-u 

mkdir /mnt/home
mount /dev/sdX /mnt/home

## Home treba da vezemo za particiju koju smo napravili
## Pre instaliranja ArchLinuxa potrebno je da sortiramo mirror da se Arch skida sa najblizih servera  

cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup 
sed -i 's/^#Server/Server/' /etc/pacman.d/mirrorlist.backup
rankmirrors -n 6 /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist
## Sada je vreme da instaliramo Arch-Linux 

pacstrap /mnt base base-devel wpa_supplicant dialog 

## Ovom komandom se instalira baza Arch-Linuxa i potrebni alati za WI-FI na laptopovima

genfstab -Lp /mnt >> /mnt/etc/fstab

## Generisanje fstab tabele

blkid /dev/sdaX >> /mnt/etc/fstab 

## Da ne bi bilo problema u buducnosti: 
## /boot koji smo vezali za boot particiju koju je Windows napravio ima svoj ID koji identifikuje njegovu lokaciju
## Nama je samo potreban UUID pa cemo umesto /dev/sdX linije u fdisk fajl-u staviti vrednost UUID ove particije
## A nju dobijamo blkid komandom

arch-root /mnt

## Ulazimo u root okruzenje da bismo pravili korisnike i dodavali pakete potrebne da bismo pokrenuli linux 

## Podesavanje jezika:

nano /etc/locale.gen

## U ovom fajlu treba izbaciti komentar kod onog jezika koji zelimo da koristimo

locale-gen

## Generisemo taj jezik

echo LANG=en_US.UTF-8 > /etc/locale.conf
export LANG=en_US.UTF-8

## Podesavanje vremena: 

ln -s /usr/share/zoneinfo/Europe/Belgrade > /etc/localtime
hwclock --systohc --utc

## Pravljenje hostname-a: 

echo vase_ime > /etc/hostname

## Sada je korisno da podesimo da Arch-Linux instalira i 32-bitne aplikacije i aplikacije iz korisnickih repozetorija
## Ovo je moguce brisanjem komentara i dodavanjem odredjenih linija u pacman.conf fajlu

nano /etc/pacman.conf

## Zatim skidamo komentare sa ovih linija 
##      [multilib]
##      Include = /etc/pacman.d/mirrorlist
## i dodajemo na kraj fajla
##      [archlinuxfr]
##      SigLevel = Never
##      Server = http://repo.archlinux.fr/$arch

## Nakom ovih izmena treba sacuvati pacman.conf fajl

pacman -Sy

## Ovom komandom se izmene pacman.conf fajla implementiraju

## Kreiranje root sifre

passwd 

## Dodavanje korisnika

useradd -m -g users -G wheel,storage,power -s /bin/bash vase_ime

passwd vase_ime

## Sifra za korisnika kog ste kreirali {vase_ime=ime korisnika kog smo kreirali}

## Treba podesiti privilegije koje ce samo root korisnik imati i o se radi na sledeci nacin

EDITOR=nano visudo

## Ovom komadnom ce se otvoriti fajl u kom treba da pronadjemo liniju %wheel ALL=(ALL) ALL i obrisati komentar
## ispod ove linije treba dodati sledecu liniju
## Defaults rootpw

## Korisno je instalirati i bash-completion koji pomaze prilikom kucanja u bash konzoli 


pacman -S bash-completion 


## Sada je vreme da instaliramo bootloader koji ce nam omoguciti da pristupimo Linuxu i Windows-u
## Imamo izbor izmedju 2 bootloader-a koji podrzavaju opciju boot-ovanja u Windows a to su Grub i Systemd-boot
## Ovde cemo koristiti Systemd-boot

bootctl --path=/boot install

## Instalacija bootloader-a je zavrsena,ali potrebno je izvrsiti odredjene konfiguracije fajlova
## Prvi fajl koji treba da konfigurisemo

nano /boot/loader/loader.conf

## U ovom fajlu treba dodati sledece linije i iskomentarisati one koje se u njemu nalaze
## default arch
## timeout 5
## editor 0

## Zatim treba konfigurisati sledeci fajl, tojest dodati opciju Arch Linux Fallback u boot manager-u

nano /boot/loader/entries/arch.conf

##       U ovaj fajl treba dodati:
##          title          Arch Linux
##          linux          /vmlinuz-linux
##          initrd         /initramfs-linux.img
##          options        root=LABEL=ARCH rw

## Prilikom instalacije Systemd-boot bootloader-a generise se i fallback kernel fajl
## Ovaj fajl nam sluzi da boot-ujemo linux sa starijim kernelom ukoliko smo nesto pokvarili
## Zato i njega treba staviti kao opciju prilikom bootovanja

cp arch.conf arch-fallback.conf
nano arch-fallback.conf

## A zatim ga treba konfigurisati, tojest dodati opciju Arch Linux Fallback u boot manager-u

##          title          Arch Linux Fallback
##          linux          /vmlinuz-linux
##          initrd         /initramfs-linux-fallback.img
##          options        root=LABEL=ARCH rw   

## Sada treba da izadjemo iz root okruzenja

exit

## Pa da podesimo krajnji izgled boot menija tojest redosled opcija u boot meniju

efibootmgr

## Ova komanda ce izlistati sve opcije za butovanje, kao i redosled boot-ovanja
## Treba da postavimo Linux Boot Manager na prvo mesto 
## Manuelno treba da podesimo redosled i to na sledeci nacin
efibootmgr -o X,X,X,X,X
## Gde su X brojevi kojima su redom podesene opcije za boot-ovanje

umount -R /mnt
reboot

## Ovim je zavrsena instalacija Arch-Linuxa, odnosno jezgra Arch-Linuxa
## Sledeci korak je instalacija drajvera za graficku karticu, drajvera za touchpad (ako instaliramo na laptopu) i zeljenog desktop-a okruzenja
## Ove instalacije cemo uraditi nakon restartovanja racunara



## Kada se racunar ponovo pokrene i u boot manager-u izaberemo Arch-Linux usli smo u Arch-Linux i mozemo da krenemo sa instaliranjem desktop-okruzenja


## Sada je potrebno da se ulogujemo kao root korisnik, ili kao korisnik kog smo napravili

## Sledeci korak je povezivanje na WI-FI ili na LAN da bismo mogli da instaliramo potrebne pakete
## Ako zelimo da se povezemo na WI-FI, u terminalu kucamo wifi-menu

## Kada smo se povezali na WI-FI kucamo

pacman -S drajver__za__graficku__karticu




