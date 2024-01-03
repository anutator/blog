---
share: "true"
title: Контейнер для сборки мобильных приложений под Android
---

Удалось сократить время на сборку приложения в 3-4 раза, заменив старый образ для сборки и подкрутив софт. На Gradle 8.3.0 не удалось ещё перейти, т.к. дальше докручивать софт программистам некогда.

Особенности контейнера для сборки мобильных приложений под Android:
 - Cобирать docker образ только на базе Ubuntu. Alpine не подходит, т.к. официально Android SDK поддерживается только в контейнерах Ubuntu.
 - В интернете для установки через **sdkmanager** пакетов они перечисляются прямо в Dockerfile, что неудобно. Список компонентов для установки лучше прописать в отдельном файле (у меня это `packages.txt`), копировать этот файл в контейнер и подставлять его путь через атрибут `--package_file`. У меня в списке софта лишний `build-tools;30.0.3`, но это из-за разработчиков, которые никак не могут рефакторить код.
 - Для разархивирования zip вместо утилиты unzip использовать утилиту **bsdtar**, которая является частью пакета **libarchive-tools**, чтобы работали все флаги, которые использует **tar**, например `--strip-components`.

```text title="packages.txt"
build-tools;34.0.0
build-tools;30.0.3
platforms;android-34
platforms;android-30
extras;android;m2repository
extras;google;google_play_services
extras;google;m2repository
add-ons;addon-google_apis-google-24
extras;m2repository;com;android;support;constraint;constraint-layout;1.0.2
emulator
platform-tools
tools
```


```dockerfile title="Dockerfile"
# C jdk11 не работает последний sdkmanager
ARG SERVICE_VERSION=7.6.2-jdk17-jammy
# На нашем Harbor зеркало hub.docker.com
FROM наш_harbor/docker/library/gradle:$SERVICE_VERSION

ARG SDK_PROXY="--proxy=http --proxy_host=наш_прокси --proxy_port=13128"
ARG ANDROID_SDK_VERSION=10406996
ENV ANDROID_HOME /opt/android-sdk
#ENV ANDROID_SDK_ROOT $ANDROID_HOME
#ENV ANDROID_SDK_HOME $ANDROID_HOME
#ENV ANDROID_SDK $ANDROID_HOME
# Важен путь к cmdline-tools, т.к. иначе sdkmanager не будет по умолчанию видеть правильный root каталог
ENV PATH $PATH:$ANDROID_HOME/cmdline-tools/latest/bin:$ANDROID_HOME/platform-tools
ENV TZ Europe/Moscow

# libarchive-tools содержит bsdtar, который может распаковывать zip с теми же опциями, как и tar
RUN apt-get update -y && apt-get -y install ca-certificates curl libarchive-tools \
 && rm -rf /var/lib/apt/lists/*

# Опционально. Здесь добавляются сертификаты Минцифры и наши.
#COPY certs/ /usr/local/share/ca-certificates/
#RUN update-ca-certificates

WORKDIR $ANDROID_HOME
RUN mkdir -p cmdline-tools/latest \
 && curl -sSL https://dl.google.com/android/repository/commandlinetools-linux-${ANDROID_SDK_VERSION}_latest.zip -o sdk-tools.zip \
 && bsdtar xvf sdk-tools.zip --strip-components=1 -C cmdline-tools/latest \
 && rm sdk-tools.zip && ls -la cmdline-tools/latest/bin && sdkmanager --version

RUN mkdir -p /root/.android \
 && touch /root/.android/repositories.cfg

# Принять лицензии перед установкой компонентов, не нужно выводить через echo для каждого компонента
# Лицензия действительна для всех стандартных компонентов в версиях, устанавливаемях из этого файла
# Нестандартные компоненты, которые требуют отдельных лицензий: MIPS system images, preview versions, GDK (Google Glass) и Android Google TV
# Принять все лицензии стандартных и нестандартных компонентов
RUN yes | sdkmanager $SDK_PROXY --licenses \
 && yes | sdkmanager $SDK_PROXY --update --channel=0

# Где проверять список версий пакетов
# https://developer.android.com/tools/releases/build-tools
# https://developer.android.com/tools/releases/platforms
# В файле со списом компонентов все их версии указывать в убывающем порядке.
COPY packages.txt .

# Полный список доступных опций: sdkmanager --list
RUN sdkmanager $SDK_PROXY --update \
 && sdkmanager --install --package_file=packages.txt $SDK_PROXY\
 && sdkmanager --list_installed

WORKDIR /home/gradle
```