all:
	make slide slideshow

slide:
	pandoc -t beamer quic.md --include-in-header=./style.tex --filter pandoc-plot -f markdown-implicit_figures -V colorlinks=true -V linkcolor=blue -V urlcolor=red -o quic.pdf
	pandoc -t beamer hls.md --include-in-header=./style.tex --filter pandoc-plot -f markdown-implicit_figures -V colorlinks=true -V linkcolor=blue -V urlcolor=red -o hls.pdf

slideshow:
	pandoc -t beamer quic.md --include-in-header=./style.tex --filter pandoc-plot -i -f markdown-implicit_figures -V colorlinks=true -V linkcolor=blue -V urlcolor=red -o quic-slideshow.pdf
	pandoc -t beamer hls.md --include-in-header=./style.tex --filter pandoc-plot -i -f markdown-implicit_figures -V colorlinks=true -V linkcolor=blue -V urlcolor=red -o hls-slideshow.pdf
