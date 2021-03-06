# Начало работы с серийной консолью

Серийная консоль — это способ получить доступ к виртуальной машине вне зависимости от состояния сети или операционной системы. Таким образом, вы можете использовать консоль, например, для устранения неисправностей виртуальных машин или при возникновении проблем с доступом через SSH.

> [!IMPORTANT]
>
> Включенный доступ к серийной консоли небезопасен: злоумышленники могут получить доступ к вашей виртуальной машине. Используйте эту инструкцию, если точно знаете, что делаете.

По умолчанию доступ к серийной консоли отключен. Вы можете предоставить доступ к серийной консоли при создании или изменении виртуальной машины. Для этого установите в метаданных параметр `serial-port-enable=1`. Значение параметра равное `0` отключает доступ. Дополнительные сведения о метаданных см. в разделе [[!TITLE]](../../concepts/vm-metadata.md).

## Перед началом работы

Перед тем, как включить доступ к серийной консоли на виртуальной машине:

1. Подготовьте пару ключей (открытый и закрытый) для SSH-доступа на виртуальную машину. Серийная консоль аутентифицирует пользователей с помощью SSH-ключей.
1. Создайте текстовый файл (например, `sshkeys.txt`) и укажите следующие данные:

    ```txt
    <имя пользователя>:<открытый SSH-ключ пользователя>
    ```

    Пример текстового файла для пользователя `yc-user`:

    ```txt
    yc-user:ssh-rsa AAAAB3Nza......OjbSMRX yc-user@example.com
    ```

    По умолчанию пользовательские SSH-ключи хранятся в каталоге `~/.ssh` этого пользователя. Получить открытый ключ можно с помощью команды `cat ~/.ssh/<имя открытого ключа>.pub`.

## Включение консоли при создании виртуальной машины из публичного образа Linux {#turn-on-for-new-instance}

Чтобы включить доступ к серийной консоли виртуальной машины:

---

**[!TAB CLI]**

[!INCLUDE [cli-install](../../../_includes/cli-install.md)]

[!INCLUDE [default-catalogue](../../../_includes/default-catalogue.md)]

1. Посмотрите описание команды CLI для создания виртуальной машины:

   ```bash
   $ yc compute instance create --help
   ```

1. Выберите один из публичных [образов](../images-with-pre-installed-software/get-list.md) на базе операционной системы Linux (например, Ubuntu).

    [!INCLUDE [standard-images](../../../_includes/standard-images.md)]

1.  Создайте виртуальную машину в каталоге по умолчанию:

    ```bash
    $ yc compute instance create \
        --name first-instance \
        --zone ru-central1-a \
        --public-ip \
        --create-boot-disk image-folder-id=standard-images,image-family=ubuntu-1604-lts \
        --metadata-from-file ssh-keys=sshkeys.txt \
        --ssh-key ~/.ssh/<имя открытого ключа>.pub \
        --metadata serial-port-enable=1
    ```

    Данная команда создаст виртуальную машину с OC Ubuntu, именем `first-instance` в зоне `ru-central1-a`, с активной серийной консолью. В ОС виртуальной машины будет автоматически создан пользователь `yc-user` с указанным открытым ключом.

    [!INCLUDE [name-format](../../../_includes/name-format.md)]

---

## Включение консоли при изменении виртуальной машины на базе ОС Linux {#turn-on-for-current-instance}

При изменении виртуальной машины имеющийся набор метаданных полностью перезаписывается набором, переданным в команде.

Чтобы не потерять метаданные:

1. Получите информацию о виртуальной машине вместе с метаданными. Все пользовательские метаданные определены в ключе `user-data`.

    ```bash
    $ yc compute instance get --full <имя виртуальной машины>
    ```

1. Скопируйте полученные данные в файл, например, `saved-userdata.yaml` и используйте его при изменении виртуальной машины.

Чтобы включить доступ к серийной консоли для ранее запущенной виртуальной машины:

---

**[!TAB CLI]**

[!INCLUDE [cli-install](../../../_includes/cli-install.md)]

[!INCLUDE [default-catalogue](../../../_includes/default-catalogue.md)]

1. Посмотрите описание команды CLI для изменения виртуальной машины:

    ```bash
    $ yc compute instance update --help
    ```

1. Получите список виртуальных машин в каталоге по умолчанию:

    [!INCLUDE [compute-instance-list](../../_includes_service/compute-instance-list.md)]

1. Выберите `ID` или `NAME` нужной машины, например `first-instance`.
1. Измените виртуальную машину в каталоге по умолчанию:

    ```bash
    $ yc compute instance update \
        --name first-instance \
        --metadata-from-file user-data=saved-userdata.yaml \
        --metadata-from-file ssh-keys=sshkeys.txt \
        --metadata serial-port-enable=1
    ```

    Данная команда изменит виртуальную машину с именем `first-instance`, активировав серийную консоль.

---

## Настройка Linux для доступа через серийный порт {#add-password}

[!INCLUDE [vm-connect-ssh-linux-note](../../../_includes/vm-connect-ssh-linux-note.md)]

Создайте локальный пароль для пользователя по умолчанию:

---

**[!TAB CLI]**

[!INCLUDE [cli-install](../../../_includes/cli-install.md)]

[!INCLUDE [default-catalogue](../../../_includes/default-catalogue.md)]

1. Получите список виртуальных машин в каталоге по умолчанию:

    [!INCLUDE [compute-instance-list](../../_includes_service/compute-instance-list.md)]

1. Выберите `ID` или `NAME` нужной машины, например `first-instance`.
1. Получите публичный IP-адрес виртуальной машины.


    ```bash
    $ yc compute instance get first-instance
    ```

    В выводе команды найдите адрес виртуальной машины в блоке `one_to_one_nat`:

    ```bash
    ...
    one_to_one_nat:
        address: <публичный IP-адрес>
        ip_version: IPV4
    ...
    ```

    Если публичного IP-адреса нет, [измените виртуальную машину](../vm-control/vm-update.md), дополнительно указав флаг `--public-ip`.

1. Подключитесь к виртуальной машине. Подробнее читайте в разделе [[!TITLE]](../vm-control/vm-connect-ssh.md#vm-connect).
1. Создайте локальный пароль. В OC Linux задать пароль можно с помощью команды `passwd`:

    ```
    sudo passwd <имя пользователя>
    ```

    Пример для пользователя `yc-user`:

    ```
    sudo passwd yc-user
    ```

1. Завершите SSH-сессию с помощью команды `exit`.

---
