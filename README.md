# Обход базовых проверок в приложении MTOP10

Выполнил Сердюков Матвей, R-220-1

При запуске приложения на главном экране выводятся сообщения о пройденных проверках, в частности:

* запущено ли приложение на эмуляторе
* присутствует ли исполняемый файл `su` в системе (проверка на `root`)
* проверка подписи приложения

Таким образом, при запуске приложения на эмуляторе, экран выглядит следующим образом:

![](./images/start.png)


## Обход проверки на эмулятор

В декомпилированном коде приложения, а именно в классе `MainActivity` присутствует функция `CheckEmulator`, результат которой влияет на показ сообщения об эмуляторе.

![](./images/check-emulator-oncreate.png)

![](./images/check-emulator-func.png)

Как можно увидеть, результат функции задаёт параметр `Visibility` у блока с текстом, сообщающем о наличии эмулятора. Эту проверку можно обойти, запатчив Smali-код так, чтобы вне зависимости от проверки условия возвращалось значение `0x8`:

![](./images/check-emulator-patch-smali.png)

В результате при пересборке приложения сообщение об эмуляторе исчезает, а также проваливается проверки подписи приложения.

![](./images/emulator-check-patched.png)

## Обход проверки на наличие `su`

Аналогичным образом выводится результат проверки на наличие в системе исполняемого файла `su`. Сама же проверка состоит в определении существования файла `su` в одной из директорий: `/system/xbin/` и `/system/bin/`.

![](./images/check-su-oncreate.png)


![](./images/check-su-func.png)

Для обхода этой проверки можно также заменить значение 0 на 8 в smali-коде функции `OnCreate()`, но я поступил по-другому — заменил пути, которая проверяет функция `checkSU()` на несуществующие:

![](./images/check-su-patch-smali.png)

В результате сообщение о проваленной проверке больше не отображается.

![](./images/su-patched.png)

## Обход проверки подписи приложения

Также в функции `OnCreate()` проверяется подпись приложения, чтобы убедиться, что оно не было модифицировано.

![](./images/check-sign-func.png)

Это ограничение можно обойти, перезаписав значение регистра `v0` в smali-коде перед проверкой условия. Таким образом, вне зависимости от результата выполнения функции `checkSign()`, всегда будет выполняться блок, сообщающий об успешном прохождении проверки.

![](./images/sign-patch-smali.png)

В результате, приложение не выводит сообщений о наличии эмулятора или рута, но при этом считает, что подпись корректна:

![](./images/finish.png)