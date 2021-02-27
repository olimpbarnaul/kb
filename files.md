# Files

### Convert DVD (VOB files)
ffmpeg -i "concat:$(echo *.VOB|tr \  \|)" combined.mp4


### Convert

#### Rotate images
for photo in *.jpg ; do convert $photo -rotate 90 $photo ; done

### Unoconv

#### Convert doc-file to pdf
unoconv -f pdf filename.docx
