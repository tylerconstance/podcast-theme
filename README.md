# Podcast Site Ghost Theme

This theme is modified from the default Casper theme, but its CSS and JS can still be trimmed down significantly.

I don't believe any JS files in the theme are being used anymore, which is nice. CSS is quite bloated still, but I can live with that for now.

## General Usage

- We'll go to understatedaudio.com/ghost and sign in, create a new post, and set the title (X: Title Name), description, and write show notes.
- Set the slug to be the episode number
- Set the publish date to the intended day
- **IMPORTANT**: Set the `Author` to the show name so the RSS feed gets generated appropriately

**NOTE**: The next couple of settings use the Facebook Card fields to set some details. This is because Ghost doesn't support custom fields (like Podcast Length and Podcast Link), so I wrote our RSS generator to use the Facebook Card data for those fields. For all intents and purposes, `Facebook Card Title` is `Podcast Length In Seconds` and `Facebook Card Description` is `Podcast URL From Libsyn`

- Go into the Facebook Card settings. Set the Facebook Title to the time of the audio file in seconds. This is so the podcast will have its time represented in the RSS feed
- Set the Facebook Description to the podcast file link from wherever it is hosted, i.e., Libsyn. Include the protocol (i.e., `https`) so the audio file will be loaded appropriately for the `<audio>` tag.


## Adding new shows

When you're ready to add a new show, create a new user. Instead of using an email address to invite a user via the Ghost admin, this can be done via a JSON import. To populate users at the start of the project, I created an `authors.json` file to import authors. Now that we have posts, the best approach may be exporting content, modifying the JSON to add a desired user, and re-importing, though this has currently been untested.

The author is the show. The author bio is the show description. To add Twitter handles to the main show page (author page), we use the [has](https://ghost.org/docs/api/v3/handlebars-themes/helpers/has/) helper and do it manually.

## RSS

Each show currently has its own RSS feed modeled after [a Ghost example](https://ghost.org/tutorials/custom-rss-feed/). I think it has to be this way since we can't really tell an RSS feed contextually which show it's supposed to be for, so we'll need to copy our RSS file and change the `primary_author` filter in the new one for every podcast we make. This *should* fill out appropriately with name, descriptions, and artwork, but give it a glance to be sure.

## Development

[Set up](https://ghost.org/docs/api/v3/ghost-cli/setup/) a new ghost site.

```
cd /var/www
mkdir -p podcast&& cd podcast
```

For development, `ghost install local`

In production, run `ghost install`.
mysql - creates a MySQL user
nginx - creates nginx config
ssl - sets up SSL with letsencrypt using acme. For most of my other sites, I'm doing this manually with certbot, but if Ghost manages this alright on its own, I won't stop it.
migrate - initializes a database
linux-user - creates a low-level linux user, which I may already have

Copy over the `content` folder and get an export of the post data.

Do theme development locally with `ghost run --development` and `npm run dev` to run the `gulp` build processes.

## Putting this on the server.

Taking this one step-by-step from local to my server.

1. Create an export of the data from Ghost admin Labs section
2. On the server, log in to mysql root with `mysql -u root -p`
3. Run `show databases;`, and if we don't have one for our podcast, do something like this:
```
CREATE DATABASE podcast_db DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
CREATE USER podcast_usr IDENTIFIED BY 'our_password_here';
GRANT ALL PRIVILEGES ON podcast_db.* TO podcast_usr@localhost IDENTIFIED BY 'our_password_here';
FLUSH PRIVILEGES;
EXIT;
```

4. On the server, `cd /var/www`
5. `mkdir podcast && cd podcast`
6. `ghost install`
    a. Enter your blog URL: `https://understatedaudio.com`
    b. Enter your MySQL hostname: (press enter and accept `localhost` default)
    c. Enter your MySQL username: `podcast_usr`
    d. Enter your MySQL password: `our_password_here`
    e. Enter your Ghost database name: `podcast_db`
    f. Do you wish to set up Nginx?: (press enter for `Y`). This creates a config at ` /var/www/podcast/system/files/understatedaudio.com.conf`
    g. Do you wish to set up SSL?: `N` (see step 7) 
    h. Enter your email (For SSL Certificate): `tyler@tylerconstance.com`
    i. Do you wish to set up Systemd?: (press enter for `Y`). This creates a config at `/var/www/podcast/system/files/ghost_understatedaudio-com.service`
    j. Do you want to start Ghost?: (press enter for `Y`).
7. run `sudo certbot` and select the domain from the list.
8. Upload the `images` folder into `content`. You may need to do a little permissions futzing, but 777 whatever you need to and run `ghost doctor` afterwards; it'll tell you what to run to fix.
9. Upload the `content/settings/routes.yaml` file. The same applies as above.
10. In the *development version* of the ghost admin, download the theme from Settings > Design. This should zip things up without `node_modules`.
11. Upload the theme (via the Ghost Admin on the server) and data export JSON file.
12. Run `ghost restart` to take in new settings.


Default theme README appears below.

----

# Casper

The default theme for [Ghost](http://github.com/tryghost/ghost/). This is the latest development version of Casper! If you're just looking to download the latest release, head over to the [releases](https://github.com/TryGhost/Casper/releases) page.

&nbsp;

![screenshot-desktop](https://user-images.githubusercontent.com/353959/66987533-40eae100-f0c1-11e9-822e-cbaf38fb8e3f.png)

&nbsp;

# First time using a Ghost theme?

Ghost uses a simple templating language called [Handlebars](http://handlebarsjs.com/) for its themes.

This theme has lots of code comments to help explain what's going on just by reading the code. Once you feel comfortable with how everything works, we also have full [theme API documentation](https://ghost.org/docs/api/handlebars-themes/) which explains every possible Handlebars helper and template.

**The main files are:**

- `default.hbs` - The parent template file, which includes your global header/footer
- `index.hbs` - The main template to generate a list of posts, usually the home page
- `post.hbs` - The template used to render individual posts
- `page.hbs` - Used for individual pages
- `tag.hbs` - Used for tag archives, eg. "all posts tagged with `news`"
- `author.hbs` - Used for author archives, eg. "all posts written by Jamie"

One neat trick is that you can also create custom one-off templates by adding the slug of a page to a template file. For example:

- `page-about.hbs` - Custom template for an `/about/` page
- `tag-news.hbs` - Custom template for `/tag/news/` archive
- `author-ali.hbs` - Custom template for `/author/ali/` archive


# Development

Casper styles are compiled using Gulp/PostCSS to polyfill future CSS spec. You'll need [Node](https://nodejs.org/), [Yarn](https://yarnpkg.com/) and [Gulp](https://gulpjs.com) installed globally. After that, from the theme's root directory:

```bash
# install dependencies
yarn install

# run development server
yarn dev
```

Now you can edit `/assets/css/` files, which will be compiled to `/assets/built/` automatically.

The `zip` Gulp task packages the theme files into `dist/<theme-name>.zip`, which you can then upload to your site.

```bash
# create .zip file
yarn zip
```

# PostCSS Features Used

- Autoprefixer - Don't worry about writing browser prefixes of any kind, it's all done automatically with support for the latest 2 major versions of every browser.
- [Color Mod](https://github.com/jonathantneal/postcss-color-mod-function)


# SVG Icons

Casper uses inline SVG icons, included via Handlebars partials. You can find all icons inside `/partials/icons`. To use an icon just include the name of the relevant file, eg. To include the SVG icon in `/partials/icons/rss.hbs` - use `{{> "icons/rss"}}`.

You can add your own SVG icons in the same manner.


# Copyright & License

Copyright (c) 2013-2020 Ghost Foundation - Released under the [MIT license](LICENSE).
