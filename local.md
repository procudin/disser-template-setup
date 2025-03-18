## Локальная сборка

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
