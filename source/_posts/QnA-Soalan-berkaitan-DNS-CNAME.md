title: "QnA: Soalan berkaitan DNS, CNAME"
date: 2015-05-19 10:02:51
permalink: qna-soalan-dns-cname.html
tags:
    - dns
    - cname
category: Devops
---

OP:-
> sedang mencari solution macam mana nak buat setting macam blogger tue.
> Develop system , then user boleh provide custom domain yang akan point pada subdomain pada web system tadi.
>
> Ada idea macam mana nak buat ?

Just set a wildcard domain *.yourdomain.com -> your ip. Each user get their own subdomain.yourdomain.com. If they want custom domain, they can set a CNAME userdomain.com -> subdomain.yourdomain.com.

On your application side, there are number of ways to handle the request.

1. Use apache virtualhost to handle each domain - tedious. For every new subdomain, you have to add new vhost config.

2. Handle the mapping for subdomain in your app - look for HOST header to determine what subdomain to handle. All subdomain being served from single virtualhost.

CNAME basically an alias. Cth kamal.page.com adalah CNAME kepada very-long-domain.aws-southeast-1.com. Bila client cuba request kamal.page.com, dns akan tau subdomain tersebut adalah CNAME kepada very-long-domain.aws-southeast-1.com jadi dia akan query very-long-domain.aws-southeast-1.com utk dapatkan ip address.

CNAME juga digunakan utk memudahkan pengurusan dns. Katakan kita ada 3 subdomain - web1.page.com, web2.page.com, web3.page.com. Kalau kita point ketiga-tiga subdomain ini terus kepada IP address (A record), maka setiap kali kita tukar IP, kita kena update ketiga-tiga subdomain tersebut. Jadi biasanya orang akan gunakan CNAME spt:-

page.com IN A 69.90.210.213
web1.page.com CNAME page.com
web2.page.com CNAME page.com
web3.page.com CNAME page.com

Manakala bila kita point subdomain kepada IP, ia akan kelihatan spt:-

page.com IN A 69.90.210.213
web1.page.com IN A 69.90.210.213
web2.page.com IN A 69.90.210.213
web3.page.com IN A 69.90.210.213

Kita dapat lihat dalam contoh di atas bila mana guna A record, setiap kali berlaku perubahan IP, kita kena update semua subdomain yang ada manakala yang guna CNAME, cuma perlu update main A record. Keburukan CNAME adalah resolving dns kepada ip mengambil masa yang lebih lama sebab client akan request subdomain terlebih dahulu, dapat record CNAME, kemudian query CNAME utk dapatkan IP.

OP:-
> Katakan saya ada domain page.com yang telah di setting name server.
> Kemudian , saya buat CNAME
> test.test.com CNAME page.com
> Adakah pada domain page.com wajib ada A Record atau Name server sudah memadai ?

Saya tak pasti apa yang dimaksudkan dengan 'Name server'. Ada beberapa jenis rekod dalam dns - A record, NS record, MX record, CNAME, TXT record etc. NS record cuma apply jika kita host dns server sendiri. Cuba fahamkan terlebih bagaimana dns berfungsi, secara ringkas:-

1. Kita daftar domain page.com dgn registrar - katakan Namecheap. Namecheap bagi kita control panel utk manage domain tersebut. Dalam control panel tersebut kita boleh nyatakan apakah dns server yang boleh resolv domain tersebut kepada IP address.

2. Katakan kita guna dns server daripada zenpipe dan zenpipe publish dns server mereka sebagai ns1.zenpipe.com. Jadi kita pun masukkan dalam control panel Namecheap ns1.zenpipe.com. Sekarang bila client cuba resolve domain page.com, dia tau dia perlu rujuk kepada ns1.zenpipe.com.

3. Zenpipe juga akan sediakan control panel utk kita manage dns record bagi page.com. Setelah login ke control panel zenpipe, kita boleh mula setup domain record utk page.com.

4. Mula2 sekali kita kena tentukan page.com nak point ke IP address mana ? Maka rekod dns kita buat permulaan mungkin seperti berikut.

page.com IN A 100.22.22.22

5. Setiap domain perlu ada sekurang-kurangnya satu A record supaya domain tu point to something.

6. Setelah set A record, mungkin kita ada setup website baru - shop.page.com. Seperti yang dibincangkan sebelum ini, oleh kerana shop.page.com turut di host di server yang sama, maka kita cuma perlu setup CNAME kepada page.com.

shop.page.com CNAME page.com

7. Ada buat website baru lagi, oleh kerana website ini agak besar, kita setup satu server baru dgn ip address 100.22.22.45. Maka kita tambah rekod dns seperti berikut:-

bigsite.page.com IN A 100.22.22.45

8. Server yang digunakan utk host bigsite.page.com turut digunakan utk host support.page.com maka kita boleh setup CNAME spt berikut:-

support.page.com CNAME bigsite.page.com

9. Setup satu website baru tapi guna service hosting kat test.test.com tapi test.com benarkan kita guna custom domain jadi boleh point custom domain tersebut kepada test.test.com:-

blog.page.com CNAME test.test.com

Akhir sekali rekod dns kita akan kelihatan spt berikut:-

page.com IN A 100.22.22.22
shop.page.com CNAME page.com
bigsite.page.com IN A 100.22.22.45
support.page.com CNAME bigsite.page.com
blog.page.com CNAME test.test.com

Nampak tak kaitan antara A record dan CNAME ?
