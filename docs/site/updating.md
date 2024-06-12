# Updating the site

The QHHOJ is under active development, so occasionally you may wish to update. This is a fairly simple process.

!>  The QHHOJ development team makes no commitment to backwards compatibility. It's possible that an update migration
    might add, change, or delete data from your install. Always back up before attempting an update. <br> <br>

First, switch to the site virtual environment, and pull the latest changes.

```
(qhhojsite) $ git pull origin master
```

Dependencies may have changed since the last time you updated, so install any missing ones now.

```
(qhhojsite) $ pip3 install -r requirements.txt
```

The database schema might also have changed, so update it.

```
(qhhojsite) $ ./manage.py migrate
(qhhojsite) $ ./manage.py check
```

Finally, update any static files that may have changed.

```
(qhhojsite) $ ./make_style.sh
(qhhojsite) $ ./manage.py collectstatic
(qhhojsite) $ ./manage.py compilemessages
(qhhojsite) $ ./manage.py compilejsi18n
```

That's it! You may wish to condense the above steps into a script you can run at a later time.
