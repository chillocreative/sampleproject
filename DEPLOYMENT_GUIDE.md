# Deployment Guide for cPanel

This guide outlines the steps to deploy the Laravel application to a cPanel shared hosting environment.

## 1. Prepare the Application Locally

1.  **Build Frontend Assets**:
    Run the following command to compile assets for production:
    ```bash
    npm run production
    ```
    *(Note: Ensure you have run `npm install` first)*

2.  **Clean Up**:
    Remove `node_modules` folder to save space (you don't need to upload it, but you CAN if you can't run npm on the server. However, standard practice is to build locally and upload `public/css`, `public/js` etc. OR upload `node_modules` if the server doesn't have Node.js. Given cPanel often lacks Node, we will assume you upload everything OR build locally and upload the artifacts. Best practice for shared hosting without Node access: **Upload everything EXCEPT `node_modules` if you already built the assets into `public`. If your app relies on runtime Node, you need Node on server. Laravel usually only needs Node for build.**)
    
    *Recommended for this app*: Build locally (`npm run production`), then upload. Do NOT upload `node_modules` unless you specifically have server-side Node dependencies for running the app (rare for standard Laravel).

3.  **Zip the Project**:
    Compress the entire project folder (excluding `.git` and `node_modules`).

## 2. Server Configuration (cPanel)

1.  **Database Setup**:
    *   Go to **MySQL Databases**.
    *   Create a new Database.
    *   Create a new User and Password.
    *   Add User to Database with **ALL PRIVILEGES**.

2.  **Upload Files**:
    *   Go to **File Manager**.
    *   Upload the `.zip` file to the root directory (usually outside `public_html` if possible, e.g., `/home/username/apps/my-app`, then symlink. BUT for simple cPanel deployment, users often upload to `public_html`).
    
    **Option A (Recommended - Secure):**
    *   Create a folder `project` in your home directory (e.g., `/home/username/project`).
    *   Upload and extract the zip there.
    *   Move the contents of `public` folder to your `public_html`.
    *   Edit `public_html/index.php`:
        Change:
        ```php
        require __DIR__.'/../vendor/autoload.php';
        $app = require_once __DIR__.'/../bootstrap/app.php';
        ```
        To:
        ```php
        require __DIR__.'/../project/vendor/autoload.php';
        $app = require_once __DIR__.'/../project/bootstrap/app.php';
        ```

    **Option B (Simple - inside public_html):**
    *   Upload everything to `public_html`.
    *   Use `.htaccess` to redirect traffic to `public/` folder OR move everything from `public/` to `public_html` and adjust paths in `index.php`.

3.  **Environment Config**:
    *   Rename `.env.example` to `.env`.
    *   Edit `.env` with your database credentials:
        ```ini
        DB_DATABASE=your_db_name
        DB_USERNAME=your_db_user
        DB_PASSWORD=your_db_password
        APP_URL=https://yourdomain.com
        APP_ENV=production
        APP_DEBUG=false
        ```

4.  **Install Dependencies (If SSH available)**:
    *   If you didn't upload `vendor`, run `composer install --optimize-autoloader --no-dev`.
    *   If you can't run composer on server, you MUST upload the `vendor` folder from your local machine. (Run `composer install --optimize-autoloader --no-dev` locally before zipping).

5.  **Application Key & Caching**:
    *   If you have SSH:
        ```bash
        php artisan key:generate
        php artisan config:cache
        php artisan route:cache
        php artisan view:cache
        ```
    *   If no SSH: Ensure `APP_KEY` is set in `.env` (copy from local or generate one). You can use a route to clear cache if needed `Route::get('/clear-cache', function() { Artisan::call('optimize:clear'); return 'Cleared!'; });`.

6.  **Storage Link**:
    *   You need to link `public/storage` to `storage/app/public`.
    *   SSH: `php artisan storage:link`
    *   No SSH: You can use a PHP script/Cron job to run the symlink command or manually create the symlink if FileManager allows.

7.  **Migrations**:
    *   Import your local database SQL dump via **phpMyAdmin**.
    *   OR run `php artisan migrate` if SSH is available.

## Checklist for Upload
- [ ] `vendor` folder (if no Composer on server)
- [ ] `public/css`, `public/js` (compiled assets)
- [ ] `.env` file configured
