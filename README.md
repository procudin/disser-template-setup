# Инструкция по настройке окружения для компиляции шаблона диссертации в LaTeX

Сссылка на шаблон - [https://github.com/AndreyAkinshin/Russian-Phd-LaTeX-Dissertation-Template](https://github.com/AndreyAkinshin/Russian-Phd-LaTeX-Dissertation-Template)

### Установка и настройка компилятора

- Скачиваем [miktex](https://miktex.org/download), посмотрел по аналогам, этот вроде самый удобный под windows
- После установки необходимо обновить компоненты:
  ![image](https://github.com/procudin/disser-template-setup/assets/20419403/68594348-8824-440a-b6ae-e009c30329e5)
- На вкладке "Packages" можно вручную установить недостающие пакеты, которые у компилятора не получилось автоматически при компиляции установить  (у меня например отсутвовал пакет "tabu"). Сюда стоит заходить только в случае ошибок в документе из-за недостающих пакетов.
![image](https://github.com/procudin/disser-template-setup/assets/20419403/31385796-7ddc-4eb7-bb43-54d6f8ff2cde)

### Установка и настройка редактора
- Скачиваем [TeXstudio для Windows 10+](https://www.texstudio.org/) [TeXstudio для Windows 8](https://github.com/texstudio-org/texstudio/releases/tag/2.12.22)
- Заходим в "Параметры" -> "Конфигурация TexStudio" -> "Компиляция" и меняем библиографию по умолчанию на "Biber".
![image](https://github.com/procudin/disser-template-setup/assets/20419403/761c1a52-b598-4e97-91c9-eb7d26efb790)
- открываем dissertation.tex и пробуем скомпилировать. В случае успеха, в логах не должно быть ошибок, в pdf должен корректно отображаться список литературы.
