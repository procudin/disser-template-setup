# Инструкция по настройке окружения для компиляции шаблона диссертации в LaTeX

Сссылка на шаблон - [https://github.com/AndreyAkinshin/Russian-Phd-LaTeX-Dissertation-Template](https://github.com/AndreyAkinshin/Russian-Phd-LaTeX-Dissertation-Template)

Прямая ссылка на скачивание - [https://github.com/AndreyAkinshin/Russian-Phd-LaTeX-Dissertation-Template/archive/refs/heads/master.zip](https://github.com/AndreyAkinshin/Russian-Phd-LaTeX-Dissertation-Template/archive/refs/heads/master.zip)

### Установка и настройка компилятора

- Скачиваем [miktex](https://miktex.org/download), посмотрел по аналогам, этот вроде самый удобный под windows
- После установки необходимо обновить компоненты:
  ![image](https://github.com/procudin/disser-template-setup/assets/20419403/68594348-8824-440a-b6ae-e009c30329e5)
- На вкладке "Packages" можно вручную установить недостающие пакеты, которые у компилятора не получилось автоматически при компиляции установить. Необходимо вручную установить пакет "tabu", т.к. он почему-то не устанавливается автоматически. 
![image](https://github.com/procudin/disser-template-setup/assets/20419403/31385796-7ddc-4eb7-bb43-54d6f8ff2cde)

### Установка и настройка редактора
- Скачиваем [TeXstudio для Windows 10+](https://www.texstudio.org/) [TeXstudio для Windows 8](https://github.com/texstudio-org/texstudio/releases/tag/2.12.22)
- Заходим в "Параметры" -> "Конфигурация TexStudio" -> "Компиляция" и меняем библиографию по умолчанию на "Biber", а компилятор меняем на "XeLaTeX".
![image](https://github.com/procudin/disser-template-setup/assets/20419403/8f605722-1a70-4e7d-ad3c-2e60b8443fc1)
- открываем dissertation.tex и пробуем скомпилировать. В случае успеха, в логах не должно быть ошибок, в pdf должен корректно отображаться список литературы.

### Примечание
- шаблон официально поддерживает 3 компилятора - pdfLaTeX, XeLaTeX, LuaTeX. При этом pdfLaTeX не поддерживает Times New Roman, так что его ценность близка к нулю в рамках этого шаблона. XeLaTeX компилирует сильно быстрее чем LuaTeX, плюс в интеренете пишут, что он сильно стабильней LuaTeX'a, поэтому в интсрукции использовал именно его.
- при смене компилятора необходимо очистить папки проекта от прошлых временных файлов (*.aux, *.toc, *.bbl, *.bcf, *.synctex.gz и прочие подобные)
- решение простых ошибок компиляции - [вот](https://github.com/AndreyAkinshin/Russian-Phd-LaTeX-Dissertation-Template/blob/master/Readme/Installation.md#%D0%BF%D1%80%D0%BE%D1%81%D1%82%D1%8B%D0%B5-%D0%BE%D1%88%D0%B8%D0%B1%D0%BA%D0%B8)

### Использование Github Actions для сборки

Для компиляции pdf можно использовать средства Github Actions. Для этого нужно добавить в репозиторий файл `.github/workflows/build.yaml`
следующего содержания:
```yaml
name: Build

on:
  pull_request: # для триггера на коммит заменить на "push" 
    branches: [ master ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: dep
        run: |
          sudo apt -y install make texlive-full ttf-mscorefonts-installer
          sudo fc-cache -fv
      - name: make
        run: |
          make
      - uses: actions/upload-artifact@v3
        with:
          name: output
          path: ./*.pdf
```

В этом файле задаются условия для запуска джобы компиляции, и шаги, из которых эта джоба состоит. Конкретно в текущем примере:
- будет запущена джоба при создании пулл-реквеста на ветку master (и соответвенно она будет запускаться при каждом изменении ветки, для которой существует открытый PR)
- установятся все необходимые зависимости
- запустится команда `make`, которая соберет абсолютно все артефакты (`synopsis`, `dissertation`, `presentation`). Если нужно собрать что-то одно, например только автореферат, то стоит заменить команду в этом шаге на `make synopsis`.
Вообще все доступные коменды можно посмотреть в `Makefile` шаблона.
- в случае успеха, все pdf файлы будут сложены в архив `output.zip` и опубликованы в артефакты сборки (вкладка `Actions` репозитория)

Советы:
- у гитхаба существует квота на GitHub Actions. Для бесплатного тарифа - 2000 минут/мес, для Github Pro - 3000 минут/мес (Github Pro можно получить студентам/преподавателям бесплатно через Github Education).
Полная компиляция одного автореферата занимает ~10мин. Диссера очевидно, что дольше. Квоты должно по плану с запасом хватать, но все же рекомендую следить за этим и не пушить 100 коммитов в день по одному (если настроен триггер на пуш). 
- пример джобы скорее всего потребутся руками как-то минорно доработать под ваше окружение, просто вслепую копипаст может не сработать. 
Как минимум, необходимо адаптировать триггер под свои ветки. Если файлы шаблона находится в некорневой директории,
то необходимо прописать относительный путь в атрибут `working-directory` шага `make` и поменять пути поиска pdf для шага upload.

