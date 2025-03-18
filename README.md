# Инструкция по настройке окружения для компиляции шаблона диссертации в LaTeX

Сссылка на шаблон - [https://github.com/AndreyAkinshin/Russian-Phd-LaTeX-Dissertation-Template](https://github.com/AndreyAkinshin/Russian-Phd-LaTeX-Dissertation-Template)

Прямая ссылка на скачивание - [https://github.com/AndreyAkinshin/Russian-Phd-LaTeX-Dissertation-Template/archive/refs/heads/master.zip](https://github.com/AndreyAkinshin/Russian-Phd-LaTeX-Dissertation-Template/archive/refs/heads/master.zip)

### Варианты компиляции

- [Локальная компиляция в Windows](local.md)
- [Github Actions](github.md)
- [Self-hosted Overleaf](selfhosted-overleaf.md)

### Примечания
- шаблон официально поддерживает 3 компилятора - pdfLaTeX, XeLaTeX, LuaTeX. При этом pdfLaTeX не поддерживает Times New Roman, так что его ценность близка к нулю в рамках этого шаблона.
XeLaTeX компилирует сильно быстрее чем LuaTeX, плюс в интеренете пишут, что он сильно стабильней LuaTeX'a, поэтому в инструкции использовал именно его.
- при смене компилятора необходимо очистить папки проекта от прошлых временных файлов (*.aux, *.toc, *.bbl, *.bcf, *.synctex.gz и прочие подобные)
- решение простых ошибок компиляции - [вот](https://github.com/AndreyAkinshin/Russian-Phd-LaTeX-Dissertation-Template/blob/master/Readme/Installation.md#%D0%BF%D1%80%D0%BE%D1%81%D1%82%D1%8B%D0%B5-%D0%BE%D1%88%D0%B8%D0%B1%D0%BA%D0%B8)
