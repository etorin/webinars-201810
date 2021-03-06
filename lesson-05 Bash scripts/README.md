### Bash, awk, sed, grep и другие

### Задание 1
написать скрипт для крона
который раз в час присылает на заданную почту 
- X IP адресов (с наибольшим кол-вом запросов) с указанием кол-ва запросов c момента последнего запуска скрипта

Накопировал WinSCP файлы
Сайт говорит
https://mrcappuccino.ru/blog/post/razbor-logov-nginx
Этот формат логов используется по умолчанию

Посылать буду с помощью sendmail
```bash
echo "Subject: hello" | sendmail -v localhost < mail.txt
```
https://blog.edmdesigner.com/send-email-from-linux-command-line/


TOP X IP адресов с указанием кол-ва запросов
Если не классифицировать запросы GET POST то в абсолютных величинах так
http://aidalinux.ru/w/Sort тут про сортировку
```bash
awk '{print $1}' access-4560-c8671a.log | uniq -c | sort -k1nr | head
```

Итого, скрипт `01.sh` выдает статистику
```bash
[root@bashscripts vagrant]# ./01.sh -f access-4560-c8671a.log -n 2 -t GET

    169 34.192.116.178
     55 79.137.51.84
[root@bashscripts vagrant]# ./01.sh -f access-4560-c8671a.log -n 2 -t POST

    184 193.201.224.225
      1 193.106.30.98
```

Ключами можно указать переменные
Отправляю вывод в mail.txt и отсылаю как указано выше.

**Вопрос: Не смог придумать, как реализовать "c момента последнего запуска скрипта".
Где хранить переменную о последнем запуске?
Придумал сравнивать время как в секундах
Нужно парсить время в логе? 
Есть команда `date -d"2014-02-14T12:30:30" +%s` преобразования времени, но в логе формат не такой**

Добавить в cron
http://blog.sedicomm.com/2017/07/24/kak-dobavit-zadanie-v-planirovshhik-cron-v-linux-unix/
```
crontab -e
0 * * * * /home/vagrant/01.sh -f access-4560-c8671a.log -n 2
```

### Задание 2
- Y запрашиваемых адресов (с наибольшим кол-вом запросов) с указанием кол-ва запросов c момента последнего запуска скрипта

Топ популярных URL c того же сайта взял
```
awk -F\" '{print $2}' access-4560-c8671a.log | awk '{print $2}' | sort | uniq -c | sort -k1nr | head -n10
```

### Задание 3
- все ошибки c момента последнего запуска

**Вопрос: Тут имеется ввиду, все типы ошибок или просто файл переслать с временным срезом??**
Если все типы ошибок - простой awk как выше по 7 полю, наверное. 
Скажите точнее что нужно увидеть, может придумаю, как это реализовать.
С моментом последнего запуска все еще не понятно.
Тоесть можно извратиться, распарсить дату в логе, перевести в секунда и в каждой строке лога сравнивать два числа. Но на сколько это глупо?

### Задание 4
- список всех кодов возврата с указанием их кол-ва с момента последнего запуска

```bash
awk '{print $9}' access-4560-c8671a.log | sort | uniq -c | sort -k1nr
```

Итого на данный момент
3:21:52
5:39 PM - 9:00 PM


### Задание 5
в письме должно быть прописан обрабатываемый временной диапазон

Пока не реализовано. Вопрос выше.

Как подсказали в чате, сохранять строку последнюю в файле и начинать с нее.
Я суть уловил, но что делать - не ясно.
Строка, с кучей спецсимволов, просто так ее не поискать `grep`-ом
Решил искать номер строки последовательным `grep`-ом

**Вопрос: Есть скрипт с функцией `strnum.sh`. Я не не понимаю, что за магия творится внутри.
Есть ненужные строки 30 32 36 38 например, где просто вывод текущего значения. Если я их убираю - все ломается - мой паровозик `grep`-ов ничего не находит. Почему?**
Так получаю строку - типа последняя
```bash
79.137.51.84 08 Nov 2018:05:19:48 +0300 GET tomcat-cms cmd.jsp HTTP 1.1 404 3652 Python-urllib 2.7 rt=0.013 uct= uht= urt= 
[root@bashscripts vagrant]# cat access-4560-c8671a.log | sed -n 18,18p > templine
```
Так вызовом скрипта я нахожу номер этой строки в общем файле - если 0 - тоесть не найдена - `tail -n +0` отработает корректно и файл будет с начала
```bash
[root@bashscripts vagrant]# ./strnum.sh templine access-4560-c8671a.log 
79.137.51.84
275
79.137.51.84
275
08
275
Nov
275
2018:05:19:48
18
+0300
18
GET
18
tomcat-cms
1
cmd.jsp
1
HTTP
1
1.1
1
The new value is 18
18
```
Да, этот скрипт нельзя просто накопировать и запустить, у меня нет обновляющегося файла
И работает, скорее всего мой паровозик не лучшим образом - но времени не хватает довести все до ума. 

### Задание 6
должна быть реализована защита от мультизапуска

Тут я накопировал из лекции и протетировал на скрипте `locking.sh`

---
Заключение

Я считаю что задание выполнено не лучшим образом, но если я начну сейчас все отлаживать - я отстану от графика сдачи домашек.
Вцелом, перечень задач считаю решенным, а на отладку, отлов пробем - нужно время.

Что сделано и что сделано криво.
Не тестирвал на кроне, только добавил в него ссылку на скрипт
Возможно, нужно поменять пути до вызываемых утилит, из за особенностей запуска из под крона
Парсинг лога и вывод значений в нужном объеме - сделано
Обрабатываемый временный диапазон.
Тут все не просто. Копирую последнюю строку из лога - убераю в ней спецсимволы, копирую слова в массив, цикром по массиву делаю `grep` поочереди, обычно остается 1 строка - первый `grep` был с номером, поэтому я знаю номер строки.
потом `tail` делает срез текущего лога в новый файл, меняем перменные в скипте и далее работаем только с новым срезанным файлом.
Нет нормальной отправкой по почте одним красивым письмом

Все замечания прошу описать, я буду с этим работать.
А пока пора переходить к следующему заданию.

4:01:58
3:15 PM - 7:25 PM