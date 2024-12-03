# Лабораторная работа №5. Хранение данных. Настройки и внешние файлы.
Выполнила: Усова Валентина, ИСП-221С
## Что делает приложение?

Это Android-приложение скачивает PDF-файлы журналов с сайта ИТМО, используя ID журнала в качестве идентификатора. После загрузки, пользователь может просмотреть или удалить скачанный файл. При первом запуске отображается краткая инструкция.

## Работа приложения

(кликабельно)
[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/4dXUo3m3PiA/default.jpg)](https://www.youtube.com/shorts/4dXUo3m3PiA)

## 1. Обзор MainActivity
![](https://github.com/nnka1/mobile_development.lab5/blob/main/photo_2024-12-03_15-37-30.jpg)

• Взаимодействие с пользователем: Обрабатывает ввод ID журнала в поле journalIdEditText и обрабатывает нажатия кнопок "Загрузить" (downloadButton), "Смотреть" (viewButton) и "Удалить" (deleteButton).

• Загрузка файлов: Использует OkHttp для асинхронной загрузки PDF-файла. Загрузка происходит в фоновом потоке.

• Обработка ответов сервера: Проверяет успешность запроса и тип содержимого, отображая сообщения об успехе, ошибках или отсутствии файла.

• Просмотр PDF-файлов: Использует FileProvider для безопасного предоставления доступа к скачанному файлу и запускает системный PDF-ридер.

• Удаление файлов: Удаляет скачанный файл с устройства.

• Хранение настроек: Использует SharedPreferences для хранения информации о том, нужно ли показывать инструкцию при запуске.

• Отображение статуса: Обновляет statusTextView для отображения текущего статуса (загрузка, завершение, ошибки).


## 2. Элементы UI

Пользовательский интерфейс MainActivity состоит из:

• EditText journalIdEditText: Поле для ввода идентификатора журнала (целое число).

• Button downloadButton: Кнопка запуска загрузки PDF-файла.

• Button viewButton: Кнопка для просмотра загруженного файла (отключена до загрузки).

• Button deleteButton: Кнопка для удаления загруженного файла (отключена до загрузки).

• TextView statusTextView: Отображает статус загрузки и сообщения об ошибках.


## 3. Загрузка файла (downloadFile() метод)

Метод downloadFile() выполняет следующие действия:

1. Получение ID журнала: Извлекает ID журнала из journalIdEditText.
   ```
       String journalId = journalIdEditText.getText().toString();
    if (journalId.isEmpty()) {
        statusTextView.setText("Введите ID журнала");
        return;
    }

   ```
2. Проверка ID: Проверяет, не пустое ли поле ввода.
3. Формирование URL: Создает URL для загрузки PDF-файла, используя полученный ID.
```
    Request request = new Request.Builder()
                        .url("https://ntv.ifmo.ru/file/journal/" + journalId + ".pdf")
                        .build();

```
4. Отправка запроса: Использует OkHttp для отправки GET-запроса к серверу. Загрузка выполняется в отдельном потоке для предотвращения блокировки UI.
```
    Response response = client.newCall(request).execute();
```
5. Обработка ответа: Проверяет код ответа (isSuccessful()) и тип содержимого (Content-Type). Обработка ошибок происходит в блоке catch.
```
    if (response.isSuccessful() && response.header("Content-Type") != null && response.header("Content-Type").startsWith("application/pdf")) {
        // ... обработка успешного ответа ...
    } else {
        runOnUiThread(() -> statusTextView.setText("Файл не найден."));
    }

```
6. Сохранение файла: Если загрузка успешна, сохраняет файл в директорию Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DOWNLOADS).
```
    try (InputStream inputStream = response.body().byteStream();
         OutputStream outputStream = new FileOutputStream(downloadedFile)) {
        byte[] buffer = new byte[4096];
        int bytesRead;
        while ((bytesRead = inputStream.read(buffer)) != -1) {
            outputStream.write(buffer, 0, bytesRead);
        }
    }

```
7. Обновление UI: Обновляет statusTextView, а также включает кнопки "Смотреть" и "Удалить". Это делается с помощью runOnUiThread, чтобы изменения были видны в основном потоке.

## 4. Просмотр и удаление файлов

• openPDF(): Открывает PDF-файл с помощью системного приложения для просмотра PDF-файлов, используя FileProvider для обеспечения безопасного доступа. Обрабатывает ActivityNotFoundException в случае отсутствия подходящего приложения.
• deleteFile(): Удаляет файл с устройства, обновляя UI. 


## 5. Обработка разрешений

Приложение запрашивает разрешение на запись во внешнее хранилище (WRITE_EXTERNAL_STORAGE).
```
private void requestStoragePermission() {
    if (ContextCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
        ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE}, REQUEST_STORAGE_PERMISSION);
    } else {
        downloadFile();
    }
}

@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    if (requestCode == REQUEST_STORAGE_PERMISSION) {
        if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
            downloadFile();
        } else {
            Toast.makeText(this, "Permission denied", Toast.LENGTH_SHORT).show();
        }
    }
}
```
## 6. Инструкция при первом запуске

При первом запуске приложения отображается диалоговое окно с инструкцией. Пользователь может отметить флажок, чтобы отключить отображение инструкции в будущем. Настройки сохраняются с помощью SharedPreferences. 


## 7. Используемые библиотеки

• OkHttp: Для отправки HTTP-запросов.
• ActivityCompat и ContextCompat: Для работы с разрешениями.

## Как собрать проект?
1. Загрузка или клонирование репозитория:
* Скачайте файлы проекта (ZIP-архив) и разархивируйте их в удобную папку или клонируйте репозиторий с помощью команды git clone [URL репозитория].

2. Открытие проекта в Android Studio:
* Откройте Android Studio.
* В главном окне выберите "Open an existing Android Studio project".
* Найдите папку, в которую вы скачали или клонировали проект, и выберите файл build.gradle.

3. Запуск приложения:
* В Android Studio, нажмите кнопку "Run" (зеленый треугольник) на панели инструментов.
* Выберите эмулятор или подключенное Android-устройство, на котором вы хотите запустить приложение.
* Дождитесь завершения сборки и запуска приложения.
