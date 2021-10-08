# summernote-file

Summernote File - Плагин для текстового редактора Summernote, который позволяет загружать картинки и различные файлы и вставлять их в текстовое поле редактора в виде ссылки на него. 

<b>Версия:</b> 1.1 от 09.10.2021<br>
<b>Предложения/Баги/Ошибки принимаются на сайте: </b> [culabra.ru](https://culabra.ru/)<br><br>



[Summernote](https://summernote.org/) сам плагин текстового редактора.

[![npm version](https://badge.fury.io/js/summernote-file.svg)](https://badge.fury.io/js/summernote-file)

_Сделан на основе плагина [summernote-audio](https://github.com/taalendigitaal/summernote-audio)._

Он может обрабатывать **изображения** (jpg, png, gif, wvg, webp), **аудио файлы** (mp3, ogg, oga), и **видео файлы** (mp4, ogv, webm) и "другие" без загрузки в base64.

Вы также можете определить свой собственный дескриптор, чтобы  **загружать эти файлы и файлы любого другого типа** на свой сервер и отображать их в Summernote.

### Классическое использование

Включите скрипт плагина после включения Summernote:

```html
<!-- include jquery, bootstrap, summernote here -->

<script type="text/javascript" src="summernote-file.js"></script>
```

### NPM

Вы можете добавить файл summernote в свой проект с помощью [npm](https://www.npmjs.com/) : npm i summernote-file


### Конфигурация

Добавьте кнопку файла на панель инструментов Summernote:

```javascript
$('.summernote').summernote({
    toolbar:[
        ['insert', ['link', 'picture', 'video', 'file']],
    ],
});
```

### Тип файла

По умолчанию плагин может обрабатывать изображения, аудио и видео файлы в **base64**. 
Чтобы обрабатывать все типы файлов, **вы должны реализовать обратный вызов (callback) onFileUpload** для их загрузки на свой сервер:


```javascript
$('.summernote').summernote({
    // Ваш код с основными настройками summernote здесь

    //Define the callback
    callbacks: {
        onFileUpload: function(file) {
            // Здесь идет ваш собственный код 
        },
    },
});
```

### Пример функции Callback для загрузки 

Вот пример Callback (с обработкой хода загрузки):


```javascript
$('.summernote').summernote({
    //Ваш код запуска summernote, согласно официальной документации.
    
    //Определение функции и свои дополнения callback
    callbacks: {
        onFileUpload: function(file) {
            uploadFile(file[0]);
        },
    },
});

function uploadFile(file, editor, welEditable) {
    var data = new FormData();
    data.append("file", file);
    $.ajax({
        url: '/filemanager/upload/', //Ваш собственный обработчик
        cache: false,
        contentType: false,
        processData: false,
        data: data,
        type: "POST",
		dataType: 'json',
		xhr: function() { //Добавляем прогресс бар при загрузке файлов
            let myXhr = $.ajaxSettings.xhr();
			$('.note-editable').after('<progress></progress>');
            if (myXhr.upload) myXhr.upload.addEventListener('progress', progressHandlingFunction, false);
            return myXhr;
        },
        success: function(url) {
			$('.summernote').summernote('insertImage', url);

			let listMimeImg = ['image/png', 'image/jpeg', 'image/webp', 'image/gif', 'image/svg'];
			let listMimeAudio = ['audio/mpeg', 'audio/ogg'];
			let listMimeVideo = ['video/mpeg', 'video/mp4', 'video/webm'];
			let elem;

			if (listMimeImg.indexOf(file.type) > -1) {
				//Изображения
				$('.summernote').summernote('insertImage', url);
			} else if (listMimeAudio.indexOf(file.type) > -1) {
				//Аудио
				elem = document.createElement("audio");
				elem.setAttribute("src", url);
				elem.setAttribute("controls", "controls");
				elem.setAttribute("preload", "metadata");
				$('.summernote').summernote('insertNode', elem);
			} else if (listMimeVideo.indexOf(file.type) > -1) {
				//Видео
				elem = document.createElement("video");
				elem.setAttribute("src", url);
				elem.setAttribute("controls", "controls");
				elem.setAttribute("preload", "metadata");
				$('.summernote').summernote('insertNode', elem);
			} else {
				//Другие типы файлов
				elem = document.createElement("a");
				let linkText = document.createTextNode(file.name);
				elem.appendChild(linkText);
				elem.title = file.name;
				elem.href = url;
				$('.summernote').summernote('insertNode', elem);
			}
		},
        error: function(data) {
            console.log(data);
        }
    });
}

function progressHandlingFunction(e) {
    if (e.lengthComputable) {
        //Изменение хода загрузки прогресс бара
		$('progress').attr({value:e.loaded, max:e.total});
    
        //Отображение в консоли информации о ходе загрузки файла
        //console.log((e.loaded / e.total * 100) + '%');

        //Сброс прогресс бара после загрузки
        if (e.loaded === e.total) {
            //Обнуляем прогресс бар после загрузки файла
			$('progress').attr('value','0.0');
    
            //Удаляем прогресс бар после загрузки файла
			$('progress').remove();
    
            //Отображение в консоли информамции по завершению загрузки
            //console.log("Загрузка завершена.");
        }
    }
}

```

### Локализация

В настоящее время поддерживает следующие языки:
* Русский
* English
* French
* Czech
