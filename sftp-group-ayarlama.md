Aşağıdaki doküman takip edilerek yapılmıştır : 

[How To Enable SFTP Without Shell Access on Ubuntu 20.04 | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-enable-sftp-without-shell-access-on-ubuntu-20-04)

### Yeni Kullanıcı (pfapp) Oluşturma

`sudo systemctl status sshd` 

`sudo su` 

`sudo adduser pfapp` 

`mkdir -p /var/sftp/uploads`

`chmod 755 /var/sftp` 

`chown root:root uploads` 

Burada bir user oluşturduk ve uploads klasörüne root yekisi veriyoruz. Üst dizinin root olması gerekiyor.

nano /etc/ssh/shhd_config bu dosyada pfapp User’ı için şunu ekliyoruz

```jsx
Match User pfapp
ForceCommand internal-sftp
PasswordAuthentication yes
ChrootDirectory /var/sftp/uploads
PermitTunnel no
AllowAgentForwarding no
AllowTcpForwarding no
X11Forwarding no

```

`systemctl restart sshd` 

### Klasör Yapısını Oluşturma

uploads klasörünün altına pares klasörünü oluşturuyoruz

`mkdir pares` 

`cd pares` 

`mkdir data` 

### Pares User oluşturma

Sonrasında pares klasörü için bir pares_user oluşturuyoruz

useradd pares

passwd pares 

nano /etc/ssh/sshd_config altına config giriyoruz

```jsx
Match User pares
ForceCommand internal-sftp
PasswordAuthentication yes
ChrootDirectory /var/sftp/uploads/pares
PermitTunnel no
AllowAgentForwarding no
AllowTcpForwarding no
X11Forwarding no
```

`systemctl restart sshd`

### Kullanıcı Grubu Oluşturma ve Kullanıcıları Eklemek

sftp-users adında bir group oluşturuyoruz ve oluşturduğumuz userları bu group’a ekliyoruz.

`groupadd sftp-users` 

`usermod -aG sftp-users pfapp`

`usermod -aG sftp-users pares_user`

`groups pares_user`

`usermod -aG root pfapp` (?)

![image.png](attachment:dd490feb-23d0-4d6a-b7be-80b7ba438688:image.png)

### Yetkilendirme Ayarları

`chmod g+rw data`

`chown pares_user:sftp-users data`

![image.png](attachment:fe94ae72-02a5-46a9-ba27-8112f918845d:image.png)

## Yeni bir kullanıcı ve folder istenirse yapılacaklar:

### 1.adım :

ziraat folder’ını oluştur ve bu folder’ı sftp-users grubuna dahil et ve 755 iznini ver. 

`mkdir ziraat 
 chown root:sftp-users ziraat/`

`chmod 755 ziraat/` 

### 2.adım :

Yeni bir user oluşturalım 

`useradd z-user`

`passwd z-user`

Bunun yerine 

`adduser z-user` 

komutu da kullanılabilir. 

### 3.adım :

Oluşturduğun user’ı sftp-users grubuna dahil edelim

`usermod -aG sftp-users z-user`

`groups z-user`

### 4.adım :

Config’i açarak z-user kullanıcısının giriş yaptığında sadece alaklaı dizini görmesi chroot ile sağlayalım.

nano /etc/ssh/sshd ye config’i ekle

```jsx
Match User z-user
ForceCommand internal-sftp
PasswordAuthentication yes
ChrootDirectory /var/sftp/uploads/ziraat
PermitTunnel no
AllowAgentForwarding no
AllowTcpForwarding no
X11Forwarding no
```

`systemctl restart sshd`

### 5.adım :

ziraat folder’ının altına z-data folder’ı oluştur. Ve bu folder’a g+rw iznini verek hem z-user hem de pfapp tarafından erişilip okunup yazılmasını sağla

`mkdir z-data`

`chown z-user:sftp-users z-data`

`chmod g+rw z-data`

Not: pfapp kullanıcı girdiğinde tüm folderları görüntüleyebilir. 

![image.png](attachment:5adeefe4-80a6-4edd-939b-fa23d5804afc:image.png)

Ancak burada bu dizine dosya oluşturamaz. Bu folderların altında bulunan data folderlarına dosya oluşturabilir ve silebilir.  İzinler aşağıdaki şekilde olacak : 

![image.png](attachment:c1c810c6-adbf-4717-a138-7f62697a99d1:image.png)

z-user giriş yaptığında ise sadece kendisi ile ilgili olan z-data folder’ını görebilir yine bu dizine yazamaz. Bir alt dizine yazabilir ve silebilir. Yani cd ile z-data’nın altına girdiğinde istediği işlemi yapmakta özgür. 

![image.png](attachment:838b58ff-eccd-4016-b979-5f394ffc1221:image.png)

Dosya izinleri farklı kullanıcılar giriş yapıp eriştiğinde bu şekilde görünecektir : 

![image.png](attachment:92b626a6-ccbd-4460-9377-4ec0b345dd9e:image.png)

### Ekstra ss’ler

![image.png](attachment:a046692e-38c1-41ff-b920-efabe5e31b0d:image.png)

![image.png](attachment:d462c13a-75a4-4f76-8655-0b5949e56be5:image.png)

![image.png](attachment:43a1c6df-501d-4350-9f66-186bc3eab4da:image.png)

![image.png](attachment:c60c88de-c41d-4763-80fe-b64258938870:image.png)
