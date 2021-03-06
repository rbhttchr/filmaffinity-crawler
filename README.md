# What is this?

You are not a real programmer if you don't do a crawler/scraper for your favorite website at least once in your life. This is my second time with [Filmaffinity](http://www.filmaffinity.com).

I wrote a scraper for it some years ago in PHP, and it kind of worked, but I wanted to try the power of [Node.js](https://nodejs.org).

This crawler takes advantage of the asynchronous paradigm and it sends simultaneous requests in parallel, so it reduces in some orders of magnitude the time to crawl the whole web.

# What do you need

Filmaffinity.com will block your IP if you do too many request so, if you want to get all their film information (more than 170 000 at this moment), some workaround is needed.

First I was thinking about using some different public proxies, but I didn't want to take care of them (if they are up or not, etc), so I'm taking advantage of [Tor](https://www.torproject.org/) and [Polipo](https://github.com/jech/polipo).

You will need to have them installed and running, either as a service or standalone.

If you want to store the data scraped you will also need to connect to a mysql server.

Finally, to run this piece of code you have to use [Node.js](https://nodejs.org).

# Steps to run

1. Create a new database:

    ```
    CREATE DATABASE filmaffinity;
    ```

2. Add a new user for that database and grant permissions:
    ```
    CREATE USER 'filmaffinity'@'localhost' IDENTIFIED BY 'filmaffinity';

    GRANT ALL PRIVILEGES ON filmaffinity.* TO 'filmaffinity'@'localhost';
    ```

3. Import database structure:
    ```
    mysql -ufilmaffinity -pfilmaffinity filmaffinity < sql/db_structure.sql
    ```

4. Install node modules:
    ```
    npm install
    ```

5. Run Tor and Polipo (or be sure that they are running as services).

6. Double check that the database name, user and password you added in the previous steps match the ones in config/parameters.ini

7. Run it!
    ```
    node crawl.js action
    ```

    You must specify one valid action:
    ```
    all: Crawls all the movies
    new: Crawls new recently added movies
    popular: Crawls most popular movies from last week
    theatres: Crawls films currently in theatres
    failed: Crawls films that previously failed to be crawled
    user_friends: Crawls friends from a user id (filmaffinity id)
    user_friends_ratings: Crawls last ratings from users friends
    user_friends_films: Crawls last films rated from friends (so those films are up to date)
    id: Crawls an specific film by id and outputs the film info (option used mostly for debug purposes)
    ```

    If you run it with the "all" action, it will start crawling [top popular films](https://www.filmaffinity.com/es/topgen.php).  All data will be populated to the database and the poster images will be downloaded to the "img" folder.

    You can take a look at crawler.log to see what is happening behind the scenes.

# Worker

There's one worker (`worker.js`) to consume jobs from RabbitMQ. Those messages are published through [Filmaffin API](https://github.com/franjid/filmaffin-api).

At the moment the worker handles `UserAddedEvent`, `UserUpdatedEvent` events from the queue. It gets and import user friends from Filmaffinity, imports last films rated by them and [sends a notification through Firebase](https://github.com/franjid/easy-firebase-notifications) when everything is done, so the user is notificated in the [App](https://github.com/franjid/filmaffin-ionic-app)
