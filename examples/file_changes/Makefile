# Print names of tracked files that have changed.

track/file_changes: track/a.txt track/b.txt track/c.txt
	touch track/file_changes

track/a.txt: files/a.txt
	echo a.txt
	touch track/a.txt

track/b.txt: files/b.txt
	echo b.txt
	touch track/b.txt

track/c.txt: files/c.txt
	echo c.txt
	touch track/c.txt

clean:
	rm track/*

distclean:
	rm -rf track
