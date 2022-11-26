all:
	make slide slideshow

slide:
	pandoc -t beamer --include-in-header=./style.tex miniconf.md -f markdown-implicit_figures -o miniconf-2022.pdf

slideshow:
	pandoc -t beamer --include-in-header=./style.tex miniconf.md -f markdown-implicit_figures -i -o miniconf-2022-slideshow.pdf

view:
	zathura --mode=presentation miniconf-2022.pdf &
