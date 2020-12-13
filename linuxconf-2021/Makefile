all:
	make slide slideshow

slide:
	pandoc -t beamer --include-in-header=./style.tex linuxconf-2021.md -o linuxconf-2021.pdf

slideshow:
	pandoc -t beamer --include-in-header=./style.tex linuxconf-2021.md -i -o linuxconf-2021-slideshow.pdf
