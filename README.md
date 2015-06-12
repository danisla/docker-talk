# Running This Presentation

Start [shellinabox](https://code.google.com/p/shellinabox/)

```
$ brew install shellinabox
$ sudo shellinaboxd -s/:LOGIN -t --no-beep
```

Create the reveal.js html

```
$ ipython nbconvert docker-talk.ipynb --to slides
```

Start the webserver with proxy to `shellinaboxd` at `/shell`

```
$ python tape.py -p 8000 -P /shell=http://localhost:4200
```
