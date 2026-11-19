**Reconnaissance** Данных о разведке нет.
index="smn_logs" source="*smn.json"
| spath
| search EventID=1
| rename EventData_* AS * 
| table UtcTime EventRecordID Computer User IntegrityLevel ParentImage Image
    CommandLine ParentCommandLine ProcessGuid ParentProcessGuid

![](attachment/Pasted%20image%2020261119191829.png)
EventID = 1 скачивание exe файла с телеграм веб приложения.


**Weaponization** **подготовка вредоносного файла** - создание вредоносной полезной нагрузки (payload) и её упаковка в доставляемый формат (например, заражённый документ, архив или исполняемый файл)
Файл называется jinglebells.exe при этом EventData_OriginalFileName: BitcoinBlackmailer.exe

**Delivery** **доставка** - файл получен через телеграм

**Exploitation**  **запуск кода на хосте** - explorer запустил скрипт. 

![](attachment/Pasted%20image%2020261119192959.png)
![](attachment/Pasted%20image%2020261119193102.png)
Запуск fondue.exe родительский процесс jingle_bells.exe  с командной строкой "C:\Windows\system32\fondue.exe" /enable-feature:NetFx3 /caller-name:mscoreei.dll -   Эта команда **запрашивает включение .NET Framework 3.5 в Windows** — среды, необходимой для работы некоторых программ


**Installation****установка и закрепление**
