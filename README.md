# Rzhevsky_hw_infrastructure - Dependencies management

## Theory [2]
[0.5] What is Docker, and how it differs from dependencies management systems? From virtual machines?\
**Ответ:** Docker это технология контейнеризации, позволяет создавать изолированные программы в отдельном процессе. Докер может функционировать в фоновом режиме и общаться через порты с внешними источниками, т.е. обладает большим функционалом, чем dependencies management systems, которые создают копии определенного окружения. В отличии от виртуальной машины, докер опирается на ядро системы и не может использовать внутри себя операционную систему отличную от той, в которой запускается.\
[0.5] What are the advantages and disadvantages of using containers over other approaches?\
**Ответ:** Преимущества - Изолированность, Стандартизация, Воспроизводимость, Откат, Масштабирование.\
Недостатки - недостаточная изолированность по сравнению с виртуальной машиной, но виртуальная машина потребляет значительно больше ресурсов.\
[0.5] Explain how Docker works: what are Dockerfiles, how are containers created, and how are they run and destroyed?\
**Ответ:** Контейнеры поднимаются (docker run) из образов, образы можно брать из репозитория (docker pull) и кастомизировать (docker build) существующие образы, создавая кастомную инструкцию по их созданию - Dockerfile. Контейнеры могут быть поднятыми (работающими - Up) и оставновленными (Exited). Некоторые контейнеры останавливаются после того как отработали, некоторые продолжают оставаться поднятыми в фоновом режиме. Чтобы удалить контейнер (docker rm) необходимо его остановить (docker stop). Если при поднятии добавить флаг (docker run --rm), то после остановки контейнер будет автоматически удален. \
[0.25] Name and describe at least one Docker competitor (i.e., a tool based on the same containerization technology).\
  **Ответ:** \
  **LXC**:\
  Unlike Docker that recommends a single process per container design pattern, the containers in LXC/LXD can run multiple processes. Additionally, docker containers are more portable because Docker efficiently abstracts resources in comparison to LXD. Lastly, Docker can run on Windows and OS X hosting environments, but LXD only supports Linux.\
**Podman**:\
  Podman does not rely on a daemon to work. Instead, Podman starts containers as child processes. It also interacts directly with the registry and with the Linux Kernel using a runtime process. It is for this reason that Podman is called a daemon-less container technology. 
The absence of a daemon improves Podman's flexibility as a container engine because it removes the dependency on a single process that could act as a point of failure that propagates to child processes causing them to fail or be orphaned. 
Podman is also significantly different from Docker in that it does not require root access. This feature provides an additional security buffer that restricts potentially dangerous processes that can manipulate crucial system settings and make the container and the contained application vulnerable. \
**Containerd**:\
Unlike Docker, Containerd however, does not handle the building of images or the creation of volumes. Interestingly, containerdis the default runtime for Docker, which is now an independent tool just like runc. This makes Containerd a handy orchestrator tool just like Kubernetes, and as a result, is one of the most popular Docker alternatives.\
<br/>
[0.25] What is conda? How it differs from apt, yarn, and others? \
 **Ответ:** Conda это система управления окружением, зависимостями и пакетами. Основная особенность conda — оригинальный менеджер разрешения зависимостей с графическим интерфейсом Anaconda Navigator, позволяющий отказаться от стандартных менеджеров пакетов (таких, как pip для Python).  От вышеперечисленных, conda отличается тем, что является бинарным менеджером пакетов и была разработана на python.

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
Содержимое RzhevskyEnv.yml .
```yml
name: rzhevsky_env
channels:
  - conda-forge
  - bioconda
  - defaults
dependencies:
  - brotli=1.0.9=hcfcfb64_8
  - brotli-bin=1.0.9=hcfcfb64_8
  - brotlipy=0.7.0=py311ha68e1ae_1005
  - bzip2=1.0.8=h8ffe710_4
  - ca-certificates=2022.9.24=h5b45459_0
  - certifi=2022.9.24=pyhd8ed1ab_0
  - cffi=1.15.1=py311h7d9ee11_2
  - charset-normalizer=2.1.1=pyhd8ed1ab_0
  - click=8.1.3=win_pyhd8ed1ab_2
  - colorama=0.4.6=pyhd8ed1ab_0
  - coloredlogs=15.0.1=pyhd8ed1ab_3
  - colormath=3.0.0=py_2
  - commonmark=0.9.1=py_0
  - contourpy=1.0.6=py311h005e61a_0
  - cryptography=38.0.4=py311h28e9c30_0
  - cycler=0.11.0=pyhd8ed1ab_0
  - dataclasses=0.8=pyhc8e2a94_3
  - expat=2.5.0=h1537add_0
  - fastqc=0.11.9=hdfd78af_1
  - font-ttf-dejavu-sans-mono=2.37=hab24e00_0
  - fontconfig=2.14.1=hbde0cde_0
  - fonttools=4.38.0=py311ha68e1ae_1
  - freetype=2.12.1=h546665d_1
  - future=0.18.2=pyhd8ed1ab_6
  - git=2.38.1=h57928b3_1
  - humanfriendly=10.0=py311h1ea47a8_4
  - idna=3.4=pyhd8ed1ab_0
  - importlib-metadata=5.1.0=pyha770c72_0
  - intel-openmp=2022.2.1=h57928b3_19741
  - jinja2=3.1.2=pyhd8ed1ab_1
  - jpeg=9e=h8ffe710_2
  - kiwisolver=1.4.4=py311h005e61a_1
  - lcms2=2.14=h90d422f_0
  - lerc=4.0.0=h63175ca_0
  - libblas=3.9.0=16_win64_mkl
  - libbrotlicommon=1.0.9=hcfcfb64_8
  - libbrotlidec=1.0.9=hcfcfb64_8
  - libbrotlienc=1.0.9=hcfcfb64_8
  - libcblas=3.9.0=16_win64_mkl
  - libdeflate=1.14=hcfcfb64_0
  - libffi=3.4.2=h8ffe710_5
  - libiconv=1.17=h8ffe710_0
  - liblapack=3.9.0=16_win64_mkl
  - libpng=1.6.39=h19919ed_0
  - libsqlite=3.40.0=hcfcfb64_0
  - libtiff=4.4.0=h8e97e67_4
  - libwebp-base=1.2.4=h8ffe710_0
  - libxcb=1.13=hcd874cb_1004
  - libzlib=1.2.13=hcfcfb64_4
  - lzstring=1.0.4=py_1001
  - m2w64-gcc-libgfortran=5.3.0=6
  - m2w64-gcc-libs=5.3.0=7
  - m2w64-gcc-libs-core=5.3.0=7
  - m2w64-gmp=6.1.0=2
  - m2w64-libwinpthread-git=5.0.0.4634.697f757=2
  - markdown=3.4.1=pyhd8ed1ab_0
  - markupsafe=2.1.1=py311ha68e1ae_2
  - matplotlib-base=3.6.2=py311h6e989c2_0
  - mkl=2022.1.0=h6a75c08_874
  - msys2-conda-epoch=20160418=1
  - multiqc=1.13=pyhdfd78af_0
  - munkres=1.1.4=pyh9f0ad1d_0
  - networkx=2.8.8=pyhd8ed1ab_0
  - numpy=1.23.5=py311h95d790f_0
  - openjdk=17.0.3=h57928b3_4
  - openjpeg=2.5.0=hc9384bd_1
  - openssl=3.0.7=hcfcfb64_1
  - packaging=21.3=pyhd8ed1ab_0
  - perl=5.32.1.1=2_h57928b3_strawberry
  - pillow=9.2.0=py311h97bb24d_3
  - pip=22.3.1=pyhd8ed1ab_0
  - pthread-stubs=0.4=hcd874cb_1001
  - pycparser=2.21=pyhd8ed1ab_0
  - pygments=2.13.0=pyhd8ed1ab_0
  - pyopenssl=22.1.0=pyhd8ed1ab_0
  - pyparsing=3.0.9=pyhd8ed1ab_0
  - pyreadline3=3.4.1=py311h1ea47a8_2
  - pysocks=1.7.1=pyh0701188_6
  - python=3.11.0=hcf16a7b_0_cpython
  - python-dateutil=2.8.2=pyhd8ed1ab_0
  - python_abi=3.11=3_cp311
  - pyyaml=6.0=py311ha68e1ae_5
  - requests=2.28.1=pyhd8ed1ab_1
  - rich=12.6.0=pyhd8ed1ab_0
  - rich-click=1.5.2=pyhd8ed1ab_0
  - rust=1.65.0=hf8d6059_0
  - rust-std-x86_64-pc-windows-msvc=1.65.0=h69312b4_0
  - setuptools=65.5.1=pyhd8ed1ab_0
  - simplejson=3.18.0=py311ha68e1ae_0
  - six=1.16.0=pyh6c4a22f_0
  - spectra=0.0.11=py_1
  - symlink-exe-runtime=1.0=hcfcfb64_0
  - tbb=2021.7.0=h91493d7_0
  - tk=8.6.12=h8ffe710_0
  - typing_extensions=4.4.0=pyha770c72_0
  - tzdata=2022g=h191b570_0
  - ucrt=10.0.22621.0=h57928b3_0
  - urllib3=1.26.13=pyhd8ed1ab_0
  - vc=14.3=h3d8a991_9
  - vs2015_runtime=14.32.31332=h1d6e394_9
  - wheel=0.38.4=pyhd8ed1ab_0
  - win_inet_pton=1.1.0=pyhd8ed1ab_6
  - xorg-libxau=1.0.9=hcd874cb_0
  - xorg-libxdmcp=1.1.3=hcd874cb_0
  - xz=5.2.6=h8d14728_0
  - yaml=0.2.5=h8ffe710_2
  - zipp=3.11.0=pyhd8ed1ab_0
  - zstd=1.5.2=h7755175_4
prefix: C:\Users\79160\.conda\envs\rzhevsky_env
```
### Docker
Создаем Dockerfile для создания образа с необходиммыми нам пакетами.
```dockerfile
FROM ubuntu:22.10
# create variable with version for STAR
ARG STAR_VERSION=2.7.10b
# create variable with necessery libraries for STAR and samtools
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
# create variable with version for samtools
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

# installing java for fastqc
RUN apt-get install -y default-jre

# downloading and uzipping FastQC, accessing rights to executable script, linking it to home/bin and removing archive
ADD http://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.11.9.zip /tmp/
RUN unzip /tmp/fastqc_v0.11.9.zip && sed -i 's/Xmx250m/Xmx2048m/' FastQC/fastqc && chmod 755 FastQC/fastqc && \
ln -s /FastQC/fastqc /home/bin/fastqc && rm -rf /tmp/fastqc_v0.11.9.zip

# downloading and installing picard
RUN git clone https://github.com/broadinstitute/picard.git && cd /picard
RUN cd picard/ && ./gradlew shadowJar

# downloading, unpacking and installing bedtools, removing acrhive afterwards
RUN wget https://github.com/arq5x/bedtools2/releases/download/v2.30.0/bedtools-2.30.0.tar.gz
RUN apt-get install -y python-is-python3
RUN tar -zxvf bedtools-2.30.0.tar.gz
RUN cd bedtools2 && \
 make  
RUN rm -rf bedtools-2.30.0.tar.gz
# adding rights to bedtools bins and coping them to /home/bin/
RUN cd bedtools2/bin && chmod a+x $(ls) && cp $(ls) /home/bin/

# downloading salmon 1.9.0
ADD https://github.com/COMBINE-lab/salmon/releases/download/v1.9.0/salmon-1.9.0_linux_x86_64.tar.gz /tmp/
RUN tar -zxvf /tmp/salmon-1.9.0_linux_x86_64.tar.gz

# installing pip and multiqc
RUN apt-get update && apt install -y python3-pip
RUN pip3 install multiqc==1.13

# continuing building salmon
RUN git clone https://github.com/oneapi-src/oneTBB.git
RUN apt-get install -y cmake
RUN cd salmon-1.9.0_linux_x86_64 && mkdir build && cd build && cmake -DFETCH_BOOST=TRUE -DTBB_INSTALL_DIR=../../oneTBB  ../../oneTBB && make && make install
# add salmon bin to docker bin
RUN cd salmon-1.9.0_linux_x86_64/bin/ && chmod a+x salmon && cp salmon /home/bin/

# this line was for removing installing libraries, but it was commented because some packages do not work without some of them
# so PACKAGES variable should be manually sorted out before removing
# RUN  apt-get --purge autoremove -y  ${PACKAGES}

# add bin dir to PATH
ENV PATH /home/bin:${PATH}


# add labels
LABEL author="Alexey Rzhevskiy" \
      maintainer="rzhevskiy01@gmail.com"

ARG BUILD_DATE 
ARG BUILD_VERSION
LABEL org.label-schema.build-date=$BUILD_DATE \
      org.label-schema.name="Bioinformatics Workbench" \
      org.label-schema.description="Boinformatics tools: fastqc=0.11.9, STAR=2.7.10b, samtools=1.16.1, picard=2.27.5, salmon=1.9.0, bedtools=2.30.0, multiqc=1.13" \
      org.label-schema.vcs-url="https://github.com/AlexeyRzhevskiy/HSE-Homeworks/blob/dependencies/README.md"\
      org.label-schema.vendor="HSE" \      
      org.label-schema.version=$BUILD_VERSION \
      org.label-schema.schema-version="1.0"

```

Собираем образ из Dockerfile и запускаем контейнер.
```bash
# building image from Dockerfile
docker build -t rzhevsky_image .

# run container from image
docker run --name rzhevsky_docker --rm -it rzhevsky_image
```
## Extra Points
### [0.25] Create an extra Dockerfile that starts from a conda base image and builds everything from your conda environment file.
Для этого пункта пришлось переустановить conda и все пакеты в новое окружение на wsl (Ubuntu) и сформировать из полученного окружения новый yml-файл, так как yml окружения сформированного на Windows не разворачивался в докере.\

yml полученный на wsl:
```yml
name: rzhevsky_env
channels:
  - bioconda
  - conda-forge
  - defaults
dependencies:
  - _libgcc_mutex=0.1=conda_forge
  - _openmp_mutex=4.5=2_gnu
  - bedtools=2.30.0=h468198e_3
  - boost-cpp=1.74.0=h6cacc03_7
  - bzip2=1.0.8=h7f98852_4
  - ca-certificates=2022.12.7=ha878542_0
  - certifi=2021.5.30=py36h5fab9bb_0
  - click=8.0.1=py36h5fab9bb_0
  - coloredlogs=15.0.1=py36h5fab9bb_1
  - colormath=3.0.0=py_2
  - commonmark=0.9.1=py_0
  - cycler=0.11.0=pyhd8ed1ab_0
  - dataclasses=0.8=pyh787bdff_2
  - expat=2.5.0=h27087fc_0
  - fastqc=0.11.9=hdfd78af_1
  - font-ttf-dejavu-sans-mono=2.37=hab24e00_0
  - fontconfig=2.14.1=hc2a2eb6_0
  - freetype=2.12.1=hca18f0e_1
  - future=0.18.2=py36h5fab9bb_3
  - humanfriendly=10.0=py36h5fab9bb_0
  - icu=69.1=h9c3ff4c_0
  - importlib-metadata=4.8.1=py36h5fab9bb_0
  - jinja2=3.0.3=pyhd8ed1ab_0
  - jpeg=9e=h166bdaf_2
  - kiwisolver=1.3.1=py36h605e78d_1
  - lcms2=2.12=hddcbb42_0
  - ld_impl_linux-64=2.39=hcc3a1bd_1
  - libblas=3.9.0=16_linux64_openblas
  - libcblas=3.9.0=16_linux64_openblas
  - libffi=3.4.2=h7f98852_5
  - libgcc-ng=12.2.0=h65d4601_19
  - libgfortran-ng=12.2.0=h69a702a_19
  - libgfortran5=12.2.0=h337968e_19
  - libgomp=12.2.0=h65d4601_19
  - libjemalloc=5.2.1=h9c3ff4c_6
  - liblapack=3.9.0=16_linux64_openblas
  - libnsl=2.0.0=h7f98852_0
  - libopenblas=0.3.21=pthreads_h78a6416_3
  - libpng=1.6.39=h753d276_0
  - libsqlite=3.40.0=h753d276_0
  - libstdcxx-ng=12.2.0=h46fd767_19
  - libtiff=4.2.0=hf544144_3
  - libuuid=2.32.1=h7f98852_1000
  - libwebp-base=1.2.4=h166bdaf_0
  - libzlib=1.2.13=h166bdaf_4
  - lzstring=1.0.4=py_1001
  - markdown=3.4.1=pyhd8ed1ab_0
  - markupsafe=2.0.1=py36h8f6f2f9_0
  - matplotlib-base=3.3.4=py36hd391965_0
  - multiqc=1.13=pyhdfd78af_0
  - ncurses=6.3=h27087fc_1
  - networkx=2.7=pyhd8ed1ab_0
  - numpy=1.19.5=py36hfc0c790_2
  - olefile=0.46=pyh9f0ad1d_1
  - openjdk=8.0.332=h166bdaf_0
  - openjpeg=2.4.0=hb52868f_1
  - openssl=1.1.1s=h0b41bf4_1
  - pandas=1.1.5=py36h284efc9_0
  - perl=5.32.1=2_h7f98852_perl5
  - pillow=8.2.0=py36ha6010c0_1
  - pip=21.3.1=pyhd8ed1ab_0
  - pygments=2.13.0=pyhd8ed1ab_0
  - pyparsing=3.0.9=pyhd8ed1ab_0
  - python=3.6.15=hb7a2778_0_cpython
  - python-dateutil=2.8.2=pyhd8ed1ab_0
  - python_abi=3.6=2_cp36m
  - pytz=2022.7=pyhd8ed1ab_0
  - pyyaml=5.4.1=py36h8f6f2f9_1
  - readline=8.1.2=h0f457ee_0
  - requests=2.13.0=py36_0
  - rich=12.6.0=pyhd8ed1ab_0
  - rich-click=1.3.0=pyhd8ed1ab_0
  - salmon=1.9.0=h7e5ed60_1
  - scipy=1.5.3=py36h81d768a_1
  - setuptools=58.0.4=py36h5fab9bb_2
  - simplejson=3.8.1=py36_0
  - six=1.16.0=pyh6c4a22f_0
  - spectra=0.0.11=py_1
  - sqlite=3.40.0=h4ff8645_0
  - star=2.7.10b=h9ee0642_0
  - tbb=2021.7.0=h924138e_0
  - tk=8.6.12=h27826a3_0
  - tornado=6.1=py36h8f6f2f9_1
  - typing_extensions=4.1.1=pyha770c72_0
  - wheel=0.37.1=pyhd8ed1ab_0
  - xz=5.2.6=h166bdaf_0
  - yaml=0.2.5=h7f98852_2
  - zipp=3.6.0=pyhd8ed1ab_0
  - zlib=1.2.13=h166bdaf_4
  - zstd=1.5.2=h6239696_4
prefix: /home/alexey/anaconda3/envs/rzhevsky_env
```
Создаем докерфайл на основе образа continuumio/anaconda3, где разворачиваем полученное окружение из yml-файла.
```dockerfile
FROM continuumio/anaconda3
LABEL authors="Alexey Rzhevsky" \
      description="Docker image containing all software requirements for the HSE project"

# install the conda environment
COPY RzhevskyEnv.yml /
WORKDIR /

# create env from yml
RUN conda env create --quiet -f RzhevskyEnv.yml && conda clean -a

# add conda env bin to PATH (instead of doing 'conda activate')
ENV PATH /opt/conda/envs/rzhevsky_env/bin:$PATH
```
Создаем образ и запускаем контейнер.
```bash 
docker build -t conda_docker .
docker run --rm --name conda_doc -it conda_docker`
