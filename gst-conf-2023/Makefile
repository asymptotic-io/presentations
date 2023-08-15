all:
	make slide slideshow

slide:
	pandoc -t beamer --include-in-header=./style.tex --filter pandoc-plot gst-conf-2023.md -f markdown-implicit_figures -o gst-conf-2023.pdf

slideshow:
	pandoc -t beamer --include-in-header=./style.tex --filter pandoc-plot gst-conf-2023.md -f markdown-implicit_figures -i -o gst-conf-2023-slideshow.pdf
