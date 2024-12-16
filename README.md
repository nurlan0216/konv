# konv
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Extract Phone Numbers to VCF</title>
    <style>
        /* Общие стили для кнопок */
        .button {
            display: inline-block;
            padding: 25px 40px;
            font-size: 20px;
            color: white;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            text-align: center;
            text-transform: uppercase;
            font-weight: bold;
            transition: background-color 0.3s, transform 0.2s;
        }

        .button:hover {
            transform: scale(1.05);
        }

        .file-button {
            background-color: #4CAF50; /* Зеленый цвет */
        }

        .file-button:hover {
            background-color: #45a049;
        }

        #extractButton {
            background-color: #007BFF; /* Синий цвет */
        }

        #extractButton:hover {
            background-color: #0056b3;
        }

        /* Стили для скрытия стандартного элемента input[type="file"] */
        #imageInput {
            display: none;
        }

        /* Стили для изображения на странице */
        #imageToShow {
            width: 100%;
            max-width: 400px; /* Уменьшенная ширина */
            margin-top: 20px;
            border: 1px solid #ddd;
            border-radius: 8px;
        }

        /* Стили для сообщения о загрузке */
        #loadingMessage {
            display: none;
            font-size: 18px;
            color: #333;
            margin-top: 20px;
        }

        /* Стили для прогресс-бара */
        #progressContainer {
            display: none;
            margin-top: 30px;
            width: 100%;
            max-width: 600px;
        }

        #progressBar {
            width: 100%;
            height: 20px;
            background-color: #f3f3f3;
            border-radius: 8px;
            border: 1px solid #ddd;
            overflow: hidden;
            position: relative;
        }

        #progressBarFill {
            height: 100%;
            background-color: #007BFF;
            width: 0%;
            transition: width 0.3s;
        }

        #progressText {
            margin-top: 10px;
            font-size: 23px;
            text-align: center;
        }

        /* Стили для отображения количества выбранных файлов */
        #fileCount {
            font-size: 23px;
            margin-top: 15px;
        }

        /* Адаптивные стили */
        @media (max-width: 600px) {
            .button {
                padding: 15px 25px;
                font-size: 16px;
            }

            #imageToShow {
                max-width: 100%;
            }

            #fileCount, #progressText {
                font-size: 18px;
            }
        }
    </style>
</head>
<body>
    <!-- Отображение изображения на странице -->
    <img id="imageToShow" src="https://static.tildacdn.com/tild3338-6138-4165-b430-616132633839/photo_53776632210303.jpg" alt="Example Image">

    <h1>Любой фото номермен бірге жібер файл ретінде сақтап берем(Скинь любое фото с номерами сохраню в виде файла)</h1>

    <!-- Кнопка выбора файла -->
    <label for="imageInput" class="button file-button">Скрин тандап ал(Выбрать картинки)</label>
    <input type="file" id="imageInput" accept="image/*" multiple>

    <!-- Отображение количества выбранных файлов -->
    <div id="fileCount">Выбрано файлов: 0</div>

    <!-- Кнопка для извлечения и сохранения номеров -->
    <button id="extractButton" class="button">Номер сақтау(Сохранить номера)</button>

    <!-- Сообщение о загрузке -->
    <div id="loadingMessage">Өткізу процесі жүріп жатыр, күтіңіз...(Процесс конвертации в процессе, пожалуйста, подождите...)</div>

    <!-- Прогресс-бар и текст для отображения процентов -->
    <div id="progressContainer">
        <div id="progressBar">
            <div id="progressBarFill"></div>
        </div>
        <div id="progressText">0%</div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/tesseract.js@4.0.2/dist/tesseract.min.js"></script>
    <script>
        document.getElementById('imageInput').addEventListener('change', function() {
            const fileCount = this.files.length;
            document.getElementById('fileCount').innerText = `Файл тандадыңыз (Выбрано файлов): ${fileCount}`;
        });

        document.getElementById('extractButton').addEventListener('click', function() {
            const files = document.getElementById('imageInput').files;
            if (files.length === 0) {
                alert('Өтініш, файлды таңдаңыз(Пожалуйста, выберите файл.)');
                return;
            }

            const phoneNumbers = [];
            const totalFiles = files.length;
            let processedFiles = 0;

            document.getElementById('loadingMessage').style.display = 'block'; // Показываем сообщение о загрузке
            document.getElementById('progressContainer').style.display = 'block'; // Показываем прогресс-бар

            const updateProgress = (percent) => {
                document.getElementById('progressBarFill').style.width = `${percent}%`;
                document.getElementById('progressText').innerText = `${Math.round(percent)}%`;
            };

            const processImage = (file, callback) => {
                const reader = new FileReader();
                reader.onload = function(event) {
                    const img = new Image();
                    img.src = event.target.result;
                    img.onload = function() {
                        Tesseract.recognize(
                            img,
                            'eng',
                            {
                                logger: m => console.log(m)
                            }
                        ).then(({ data: { text } }) => {
                            const phoneRegex = /(\+?\d{1,3}[-.\s]?)?\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}/g;
                            const matches = text.match(phoneRegex);
                            if (matches) {
                                phoneNumbers.push(...matches);
                            }
                            callback();
                        }).catch(error => {
                            console.error('Error recognizing text:', error);
                            callback();
                        });
                    };
                };
                reader.readAsDataURL(file);
            };

            const processFiles = (index) => {
                if (index < totalFiles) {
                    processImage(files[index], () => {
                        processedFiles++;
                        const progressPercent = (processedFiles / totalFiles) * 100;
                        updateProgress(progressPercent);
                        processFiles(index + 1);
                    });
                } else {
                    document.getElementById('loadingMessage').style.display = 'none'; // Скрываем сообщение о загрузке
                    document.getElementById('progressContainer').style.display = 'none'; // Скрываем прогресс-бар
                    if (phoneNumbers.length > 0) {
                        saveAsVcf(phoneNumbers);
                    } else {
                        alert('No phone numbers found.');
                    }
                }
            };

            processFiles(0);
        });

        function saveAsVcf(phoneNumbers) {
            let vcfContent = `BEGIN:VCARD\nVERSION:3.0\n`;
            phoneNumbers.forEach((number) => {
                vcfContent += `TEL;TYPE=CELL:${number}\n`;
            });
            vcfContent += `END:VCARD`;

            const blob = new Blob([vcfContent], { type: 'text/vcard' });
            const link = document.createElement('a');
            link.href = URL.createObjectURL(blob);
            link.download = 'contacts.vcf';
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);

            // Перенаправление на другую страницу после скачивания
            setTimeout(() => {
                window.location.replace('http://business-school-amanat.kz/vzlom'); // Замените на нужный URL
            }, 1000); // Задержка в 1 секунду для завершения скачивания
        }
    </script>

    <!-- Ссылки для связи через WhatsApp и Instagram -->
    <div style="text-align: center; margin-top: 50px;">
        <a href="https://wa.me/77776020216" target="_blank" style="display: inline-block; margin: 0 20px;">
            <img src="https://upload.wikimedia.org/wikipedia/commons/5/5e/WhatsApp_icon.png" alt="WhatsApp" style="width: 40px; height: 40px;">
        </a>
        <a href="https://instagram.com/_nurlan0216_" target="_blank" style="display: inline-block; margin: 0 20px;">
            <img src="https://upload.wikimedia.org/wikipedia/commons/a/a5/Instagram_icon.png" alt="Instagram" style="width: 40px; height: 40px;">
        </a>
    </div>
</body>
</html>
