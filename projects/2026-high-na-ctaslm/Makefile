TEX=main.tex
OUTDIR=../build

pdf:
	mkdir -p build
	cd tex && latexmk -pdf -outdir=$(OUTDIR) -interaction=nonstopmode -halt-on-error -file-line-error $(TEX)

clean:
	cd tex && latexmk -C -outdir=$(OUTDIR) $(TEX)
	rm -rf build
