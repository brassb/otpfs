COMPILER = gcc
SRC_FILES = otpfs.c

build: $(SRC_FILES)
	$(COMPILER) $(SRC_FILES) -o otpfs -lcrypto `pkg-config fuse --cflags --libs`
	cat README_usage.txt

clean:
	rm otpfs
