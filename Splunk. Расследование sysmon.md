Пройдемся по EventID.
index="system_logs" | rename Event.System.EventID AS EventID  Event.EventData.* AS *  - переименовали длинное название поля.

index="system_logs" | rename Event.System.EventID AS EventID  Event.EventData.* AS * | *search EventID=1* - начнем с рассмотрения всех событий с id 1 - запуск процесса. 
index="system_logs" | rename Event.System.EventID AS EventID  Event.EventData.* AS * | search EventID=1 *| table UtcTime User IntegrityLevel ParentImage Image CommandLine ParentCommandLine ProcessGuid ParentProcessGuid*  - выведем результат в таблицу 
index="system_logs" | rename Event.System.EventID AS EventID  Event.EventData.* AS * | search EventID=1 | table UtcTime User IntegrityLevel ParentImage Image CommandLine ParentCommandLine ProcessGuid ParentProcessGuid *| sort 0 UtcTime* - выстраивает все результаты по времени
