# Rzhevsky_hw_infrastructure - remote servers

## Theory [2]

* [0.4] What are [computer ports](https://www.cloudflare.com/learning/network-layer/what-is-a-computer-port/) on a high level? How many ports are there on a typical computer? \
 Порты это "адреса", по которым происходит соединение трафика с подходящими для него программами на машине. Порты являются software-based и управляются операционной системой. Количество портов ограничено с учётом 16-битной адресации (2^16 = 65536, начальный — "0", последний - "65535").
* [0.4] What is the difference between http, https, ssh, and other protocols? In what sense are they similar? Name default ports for several data transfer protocols. \
 HTTP — это протокол, в котором описаны правила передачи данных веб-страницы и получения сервером информации, которую пользователь ввёл на сайте. HTTPS — это тот же протокол, но с надстройкой безопасности (SSL/TLS). По HTTP информация передаётся в обычном виде, а по HTTPS — в зашифрованном. SSH - это протокол удаленной работы, импользуемый для безопасного соединения с удаленным сервером и выполнения на нем команд. SSH использует network tunnels, HTTPS использует цифровые сертификаты. Аутентификация SSH осуществляется через верификацию сервера, генерацию ключа и верификацию пользователя сервером. HTTPS осуществляет аутентификацию через обмен цифровыми сертификатами. Общее между SSH и HTTPS то, что они оба являются протоколами зашифрованного соединения.\
Порты для протоколов обмена: \
20-21: FTP \
22: SSH \
25: SMTP \
80: HTTP \
443: HTTPS \
587: Modern, secure SMTP that uses encryption
* [0.4] Explain briefly: (1) what is IP, (2) what IPs are called 'white'/public, (3) and what happens when you enter 'google.com' into the web browser. 
 1) The Internet Protocol (IP) is a protocol, or set of rules, for routing and addressing packets of data so that they can travel across networks and arrive at the correct destination. IP information is attached to each packet, and this information helps routers to send packets to the right place. 
 2) A public IP address is the primary address associated with your whole network. While each connected device has its own IP address, they are also included within the main IP address for your network. Your public IP address is the address that all the devices outside your internet network will use to recognize your network. White IP
 3) You type a 'google.com' in your browser and press Enter -> Browser looks up IP address for the domain -> Browser initiates TCP connection with the server via corresponding ports -> Browser sends the HTTP request to the server -> Server processes request and sends back a response -> Browser renders the content.
* [0.4] What is Nginx? How does it work on the high level? List several alternative web servers.
 Nginx is an open-source web server that, since its initial success as a web server, is now also used as a reverse proxy, HTTP cache, and load balancer.\
Nginx is built to offer low memory usage and high concurrency. Rather than creating new processes for each web request, Nginx uses an asynchronous, event-driven approach where requests are handled in a single thread. With Nginx, one master process can control multiple worker processes.\
nginx alternatives are Apache HTTP Server, Caddy, lighttpd and Varnish.
* [0.4] What is SSH, and for what is it typically used? Explain two ways to authenticate in an SSH server in detail. \
 SSH is a network protocol that gives users a secure way to access a computer over an unsecured network. SSH mostly used for maintain remote servers by system administrators and file transfers.\
 There are two ways for an authentication:\
 **Password-based authentication**\
In password-based authentication, after establishing secure connection with remote servers, SSH users usually pass on their usernames and passwords to remote servers for client authentication. These credentials are shared through the secure tunnel established by symmetric encryption. The server checks for these credentials in the database and, if found, authenticates the client and allows it to communicate. \
**Public key-based authentication**\
In Public key-based authentication, after the client establishes a connection with the remote server, the client informs the server of the key pair it would like to authenticate itself with. The server verifies the existence of this key pair in its database and then sends an encrypted message to the client. The client decrypts the message with it’s private key and generates a hash value which is sent back to the server for verification. The server generates its own hash value and compares it with the one sent from the client. When both the hash values match, the server is convinced of the client’s authenticity and allows it to communicate with the server.


## Problem [6.5]
Создаем ssh ключ на локальной машине:
```bash
ssh-keygen -t ed25519
```
Создаем виртуальную машину на Яндекс.Облаке с публичным адресом 158.160.52.103 и прописываем ей публичную часть нашего ssh-ключа. \
Подключаемся к виртуальной машине через ssh c помощью следующей команды:
```bash
ssh rzhevsky@158.160.52.103
```
Устанавливаем пакет samtools.
```bash
 sudo apt-get update -y
 sudo apt-get install -y samtools
```
Скачиваем и индексируем fasta и GFF3.
```bash
# downloading
wget https://ftp.ensembl.org/pub/release-108/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz
wget https://ftp.ensembl.org/pub/release-108/gff3/homo_sapiens/Homo_sapiens.GRCh38.108.gff3.gz

# unpacking
gzip -d Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz
gzip -d Homo_sapiens.GRCh38.108.gff3.gz

# indexing fasta via samtools faidx
samtools faidx Homo_sapiens.GRCh38.dna.primary_assembly.fa

# installing tabix for GFF3 indexing
sudo apt-get install -y tabix

# sorting gff3, packing to BGZF format and writing result in new file 
(grep "^#" Homo_sapiens.GRCh38.108.gff3; grep -v "^#" Homo_sapiens.GRCh38.108.gff3 | sort -t"`printf '\t'`" -k1,1 -k4,4n) | bgzip > sorted.Homo_sapiens.GRCh38.108.gff3.gz

# indexing sorted file with tabix
tabix -p gff sorted.Homo_sapiens.GRCh38.108.gff3.gz
```

Скачиваем BED файлы:
```bash
#CHip-seq_TMF1
wget -O Chipseq_HEPG2_TMF1.bed.gz "https://www.encodeproject.org/files/ENCFF874ZCC/@@download/ENCFF874ZCC.bed.gz" 

#CHip-seq_BRF2
wget -O Chipseq_HEPG2_BRF2.bed.gz "https://www.encodeproject.org/files/ENCFF121HYT/@@download/ENCFF121HYT.bed.gz" 

#NR3C1 CHIP-seq
wget -O Chipseq_HEPG2_JDP2.bed.gz "https://www.encodeproject.org/files/ENCFF018HXI/@@download/ENCFF018HXI.bed.gz" 

# ATAC-seq
wget -O ATACseq_HepG2.bed.gz "https://www.encodeproject.org/files/ENCFF438JMM/@@download/ENCFF438JMM.bed.gz"

# unpacking
gunzip Chipseq_HEPG2_TMF1.bed.gz Chipseq_HEPG2_BRF2.bed.gz Chipseq_HEPG2_JDP2.bed.gz ATACseq_HepG2.bed.gz

# removing "chr" from BED files
sed 's/^chr\|%$//g' Chipseq_HEPG2_TMF1.bed > cleaned.Chipseq_HEPG2_TMF1.bed
sed 's/^chr\|%$//g' ATACseq_HepG2.bed > cleaned.ATACseq_HepG2.bed
sed 's/^chr\|%$//g' Chipseq_HEPG2_BRF2.bed > cleaned.Chipseq_HEPG2_BRF2.bed
sed 's/^chr\|%$//g' Chipseq_HEPG2_JDP2.bed > cleaned.Chipseq_HEPG2_JDP2.bed

# sorting and indexing
sort -V -k1,1 -k2,2 cleaned.Chipseq_HEPG2_TMF1.bed | bgzip > sorted.Chipseq_HEPG2_TMF1.bed.gz
tabix -s 1 -b 2 -e 3 sorted.Chipseq_HEPG2_TMF1.bed.gz
sort -V -k1,1 -k2,2 cleaned.ATACseq_HepG2.bed | bgzip > sorted.ATACseq_HepG2.bed.gz
tabix -s 1 -b 2 -e 3 sorted.ATACseq_HepG2.bed.gz
sort -V -k1,1 -k2,2 cleaned.Chipseq_HEPG2_BRF2.bed | bgzip > sorted.Chipseq_HEPG2_BRF2.bed.gz
tabix -s 1 -b 2 -e 3 sorted.Chipseq_HEPG2_BRF2.bed.gz
sort -V -k1,1 -k2,2 cleaned.Chipseq_HEPG2_JDP2.bed | bgzip > sorted.Chipseq_HEPG2_JDP2.bed.gz
tabix -s 1 -b 2 -e 3 sorted.Chipseq_HEPG2_JDP2.bed.gz
```
Устанавливаем JBrowse, nginx и редактируем nginx.conf:
```bash
# installing pre-requisites
sudo apt install build-essential zlib1g-dev
sudo apt install nginx
sudo apt install npm
sudo apt install genometools

# installing jbrowse CLI
npm install -g @jbrowse/cli

# check version
jbrowse --version

# creating new jbrowse repository
jbrowse create /mnt/JBrowse

# editing nginx.config
sudo vi /etc/nginx/nginx.conf

# reloading nginx
sudo systemctl restart nginx

# check nginx status
 sudo systemctl status nginx
```

Добавляем проиндексированные файлы в jbrowse.
```bash
# adding fasta file
sudo jbrowse add-assembly Homo_sapiens.GRCh38.dna.primary_assembly.fa --load copy --out /mnt/JBrowse

# adding gff3 file
sudo jbrowse add-track sorted.Homo_sapiens.GRCh38.108.gff3.gz --load copy --out /mnt/JBrowse

# adding BED files to jbrowse
sudo jbrowse add-track sorted.Chipseq_HEPG2_TMF1.bed.gz --load copy --out /mnt/JBrowse
sudo jbrowse add-track sorted.ATACseq_HepG2.bed.gz --load copy --out /mnt/JBrowse
sudo jbrowse add-track sorted.Chipseq_HEPG2_BRF2.bed.gz --load copy --out /mnt/JBrowse
sudo jbrowse add-track sorted.Chipseq_HEPG2_JDP2.bed.gz --load copy --out /mnt/JBrowse

# add indexing 
sudo jbrowse text-index --out //mnt/JBrowse
```
Теперь JBrowser2 доступен на нашей виртуальной машине по адресу http://158.160.52.103/jbrowse/ вместе с нашими файлами.
Прикладываю [ссылку на демонстрационную сессию](http://158.160.52.103/jbrowse/?session=share-B4F3d-rMs8&password=72Wg8) нашего проекта, демонстрирующую выбранный  фрагмент.
