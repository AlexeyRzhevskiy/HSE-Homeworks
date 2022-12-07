# Rzhevsky_hw_infrastructure

## Theory [2]
[0.5] What is Docker, and how it differs from dependencies management systems? From virtual machines?\
**Ответ:** Docker это технология контейнеризации, позволяет создавать изолированные программы в отдельном процессе. Докер может функционировать в фоновом режиме и общаться через порты с внешними источниками, т.е. обладает большим функционалом, чем dependencies management systems, которые создают копии определенного окружения. В отличии от виртуальной машины, докер опирается на ядро системы и не может использовать внутри себя операционную систему отличную от той, в которой запускается.\
[0.5] What are the advantages and disadvantages of using containers over other approaches?\
**Ответ:** Преимущества - Изолированность, Стандартизация, Воспроизводимость, Откат, Масштабирование.\
Недостатки - недостаточная изолированность по сравнению с виртуальной машиной, но виртуальная машина потребляет значительно больше ресурсов.\
[0.5] Explain how Docker works: what are Dockerfiles, how are containers created, and how are they run and destroyed?\
**Ответ:** Контейнеры поднимаются (docker run) из образов, образы можно брать из репозитория (docker pull) и кастомизировать (docker build) существующие образы, создавая кастомную инструкцию по их созданию - Dockerfile. Контейнеры могут быть поднятыми (работающими - Up) и оставновленными (Exited). Некоторые контейнеры останавливаются после того как отработали, некоторые продолжают оставаться поднятыми в фоновом режиме. Чтобы удалить контейнер необходимо его остановить (docker stop). Если при поднятии добавить флаг (docker run --rm), то после остановки контейнер будет автоматически удален. \
[0.25] Name and describe at least one Docker competitor (i.e., a tool based on the same containerization technology).\
  **Ответ:** LCX, podmod, containerd.\
[0.25] What is conda? How it differs from apt, yarn, and others? \
Conda это система управления окружением, зависимостями и пакетами. От вышеперечисленных, conda отличается тем, что была разработана на python.

## Problem [6.5]
### Anaconda
Создаем окружение и устанавливаем в него необходимые пакеты.
```bash
# create new environment
conda create -n rzhevsky_env

# add bioconda channels to condarc file, from lowest to highest priority
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge

# command to avoid cryptic errors when tryng to install
conda config --set channel_priority strict

# install packages
conda install -n rzhevsky_env fastqc=0.11.9 multiqc=1.13 star=2.7.10b samtools=1.16.1 picard=2.27.5 salmon=1.9.0 bedtools=2.30.0

# activate environment for conveniency
conda activate rzhevsky_env

# install RUST for reat package, REAT has no Python interface
conda install -c conda-forge rust

# install REAT commit ee19bc928badd227975410a6b8b715c0e03bd4ab
cargo install --git https://github.com/alnfedorov/reat --rev ee19bc928badd227975410a6b8b715c0e03bd4ab
```

Из всех пакетов установились только fastqc и multiqc. Проблема в том, что у меня машина на **Windows**, а на сайте bioconda сказано - NOTE: Bioconda supports only 64-bit Linux and macOS. Для установки этих библиотек необходимо, чтобы появилась их поддержка для Windows.\
Аналогично в документации REAT указано - The target platform for REAT is an x86-64 computer running Linux; working on Windows or ARM is not guaranteed.\
Экспортируем в файл условия для создания окружения и создаем из этого файла окружение с другим именем.
```bash
# in activated environment export itself to file
conda env export > RzhevskyEnv.yml

# rebuilt environment with other name from RzhevskyEnv.yml file
conda env create --name rzhevsky_env_2 -f RzhevskyEnv.yml
```
### Docker
Создаем Dockerfile для создания образа с необходиммыми нам пакетами.
```vim
FROM ubuntu

ARG STAR_VERSION=2.7.10b

ENV PACKAGES gcc g++ make wget zlib1g-dev unzip libncurses5-dev libbz2-dev liblzma-dev \
libcurl4-gnutls-dev libssl-dev perl bzip2 gnuplot ca-certificates gawk git ant

RUN set -ex

# downloading and installing packages for installing and compiling libraries
RUN apt-get update && \
    apt-get install -y --no-install-recommends ${PACKAGES} && \
    apt-get clean && \
    g++ --version && \
    cd /home && \
# downloading, unpacking, making STAR package
    wget --no-check-certificate https://github.com/alexdobin/STAR/archive/${STAR_VERSION}.zip && \
    unzip ${STAR_VERSION}.zip && \
    cd STAR-${STAR_VERSION}/source && \
    make STARstatic && \
# creating dir for executive and copy it to this dir
    mkdir /home/bin && \
    cp STAR /home/bin && \
    cd /home && \
# removing archive
    'rm' -rf STAR-${STAR_VERSION}

ARG SAMTOOLSVER=1.16.1

# downloading, unpacking, removing archive, making and installing samtools
RUN wget https://github.com/samtools/samtools/releases/download/${SAMTOOLSVER}/samtools-${SAMTOOLSVER}.tar.bz2 && \
 tar -xjf samtools-${SAMTOOLSVER}.tar.bz2 && \
 rm samtools-${SAMTOOLSVER}.tar.bz2 && \
 cd samtools-${SAMTOOLSVER} && \
 ./configure && \
 make && \
 make install && \
 mkdir /data

# RUN  apt-get --purge autoremove -y  ${PACKAGES}

ENV PATH /home/bin:${PATH}
```

Собираем образ из Dockerfile и запускаем контейнер.
```bash
# building image from Dockerfile
docker build -t rzhevsky_image .

# run container from image
docker run --name rzhevsky_docker --rm -it rzhevsky_image
```
