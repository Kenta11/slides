DOCUMENT = index.html

.PHONY: all clean
all: $(DOCUMENT)

%.html: %.md custom.css
	pandoc -f markdown+emoji $< \
	       -t revealjs -o $@ \
	       --template=template.html \
	       -s -c custom.css \
	       --slide-level=2

clean:
	rm -f $(DOCUMENT)
