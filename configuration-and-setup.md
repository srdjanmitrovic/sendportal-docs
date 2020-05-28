# Configuration & Setup

SendPortal comes packaged with a command-line setup utility that will get you up and running in no time. It will attempt to automatically configure SendPortal and create a default user account and workspace for administrative purposes.

## Setup Command

In the SendPortal installation's root directory, run the following command:

```bash
php artisan sp:setup
```

The command will prompt you for confirmation on most steps. If this is the first time running setup for an installation, you should answer yes to the prompts in order to ensure that SendPortal is completely set up as expected.

You can safely run the command multiple times if needed.

Additionally, if you let the setup command create the `.env` configuration file, you will not be able to continue until you manually specify the database configuration settings in that file. As such, you will need to add your database configuration at this stage and then run the setup command again to continue. Database configuration instructions can be found in the Setting Up Database Connection section of Manual Configuration below.

## Manual Configuration

### Creating Configuration File

SendPortal's configuration is handled through the`.env` file. By default, this file does not exist. To create it manually, you will need to clone the included `.env.example`, rename it to `.env` and make changes as necessary.

> Any keys that are set in the `.env` file will be used, even if they are blank. If you do not wish to actively set a key (e.g. known defaults are what you want), you should remove it from your `.env` file, rather than leave it blank.

### Key Generation

If you did not run the setup command, or if for some reason the `APP_KEY` value is empty, you will need to create an encryption key. This is used by SendPortal to apply encryption to things like user sessions.

To generate a new key, you should run the following command:

```bash
php artisan key:generate
```

> Key generation can be run again, overwriting the previously set key. Doing so will invalidate any sessions or make any stored encrypted data inaccessible. You should not generate a new key unless absolutely necessary.

### Database Connection

In order for SendPortal to connect to your database, you must set the database configuration values in the `.env` file.

Firstly, you need to specify what type of database you are using by setting the `DB_CONNECTION` value to either `mysql` for a MySQL database or `pgsql` for a PostgreSQL database.

Secondly, you need to set the connection details for your database installation. The following values need to be set:

- `DB_HOST` – This is the host of your database, e.g. `127.0.0.1` for a local installation
- `DB_PORT` - The port SendPortal should use to talk to your database
- `DB_DATABASE` – The database SendPortal should use to store its data
- `DB_USERNAME` – The username SendPortal will use to authenticate itself with your database
- `DB_PASSWORD` – The password SendPortal will use to authenticate itself with your database

### Database Migrations

To set up the database schema, migrations must be run. Migrations are instructions an application uses to configure database schema, running in sequence from beginning to end in order to ensure that the database is set up as the application expects it to be.

> Do not make custom modifications to the database yourself. Any database changes that SendPortal requires should be accomplished through the running of migrations.

> Before running migrations, ensure that you have correctly configured your database connection, as schema changes will be made.

The included command-line setup command will run migrations for you (after a prompt), but you can run migrations manually using the following command:

```bash
php artisan migrate
```

### Publishing Vendor Files

Run the following command to publish the config, views, languages and assets from SendPortal to your project:

```bash
php artisan vendor:publish --provider=Sendportal\\Base\\SendportalBaseServiceProvider
```

### Workspaces & Users

If you do not use the setup command to create a workspace and user with which to administer SendPortal, you will need to go through the web interface registration process.

For this to function, you need to enable registration by adding the `SENDPORTAL_REGISTER=true` parameter to your `.env` file and additional email configuration will be necessary. In particular, the Email setup specified in the Additional Configuration section must be completed in order for registration to work.

The same configuration above is necessary for the new user invitation process to be working.

## Additional Configuration

### Cron Jobs

SendPortal makes use of regular background tasks and it is therefore essential to create a cron job to run every minute:

```bash
* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1
```

Please refer to the Laravel documentation on [Task Scheduling](https://laravel.com/docs/scheduling) for further information.

### Message Sending & Queues

SendPortal sends email messages using a queue system. The queue can be processed synchronously or asynchronously. Asynchronous queues can be handled via your primary database or via redis.

You can specify which queue to use in the `QUEUE_CONNECTION` parameter in the `.env` file. This should be set to one of `sync`, `database` or `redis`, depending on your requirements. This configuration applies to all messages sent in SendPortal and cannot be changed on a per-user, per-workspace or per-provider basis.

#### Sync

The synchronous queue runs any queued jobs as they are requested, requiring the user to wait until the job has been completed before any further action can be taken.

This has the advantage of being simple and requiring no additional configuration. However, as this does not scale well, this approach is only recommended for relatively small message sending requirements.

To use the synchronous queue, you only need to set the `QUEUE_CONNECTION` to `sync`, and no further configuration is required.

#### Database

Processing asynchronous queues via the database is considered a middle option. Running an asynchronous queue avoids the issues that come up using a synchronous queue, as messages can be processed without blocking further action being taken by the user, and will work until all jobs are completed regardless of how long it takes.

The advantage of using the database for running the asynchronous queue is that it requires no extra services to be run, as it uses the primary database (MySQL or Postgres) that you have already configured for SendPortal.

The main disadvantage is that under heavy workloads the queue can negatively affect the performance of the database.

As such, the database queue is recommended for small to medium sized mailing lists.

To use the database driver, you must first set the `QUEUE_CONNECTION` to `database` in your `.env` file. You also need to run the following commands, which will create a new `jobs` table in your database, which that will be used to manage the queue.

```bash
php artisan queue:table
php artisan migrate
```

#### Redis

Redis is the recommended solution for running medium to large mailing lists.

You will of course need an installation of redis on your server. You will then need to set the `QUEUE_CONNECTION` to `redis` and set the following configuration values in your `.env` file:

- `REDIS_HOST`
- `REDIS_PASSWORD`
- `REDIS_PORT`

### Running Redis Queues With Laravel Horizon

SendPortal bundles [Laravel Horizon](https://laravel.com/docs/7.x/horizon) as an easy way to run and manage redis queues.

Configuration for the queues necessary to run SendPortal is already included. In order to use Horizon as your queue manager you simply need to run the following command:

```
php artisan horizon
```

When using Horizon in production, you should consider using a service to ensure the queue runner restarts if it fails for any reason. The Horizon documentation has a [guide](https://laravel.com/docs/7.x/horizon#deploying-horizon) on how to use Supervisor to do this.

#### Autoscaling

The configuration for Horizon included with SendPortal allows autoscaling of queue workers. By default, webhooks received and messages sent via the queue each have a minimum of 2 processes running, and a maximum of 10 or 20. If these values do not suit your requirements, they can be adjusted in the `config/horizon.php` file—in particular `supervisor-2` and `supervisor-3`—using the `minProcesses` and `maxProcesses` values.

### User Management Email

In order to use user management functionality (for example, inviting new users or password resets) in SendPortal, it is necessary to set up an email service that SendPortal can use to send the messages.

> If you are not going to be inviting any other users or team members to your SendPortal installation, then this section can be ignored.

> There is no relationship between SendPortal's internal mail configuration and any Email Services that are configured for a workspace.

> Make sure the registration functionality is enabled by the `SENDPORTAL_REGISTER=true` parameter.

You first need to set `MAIL_MAILER` to your chosen service. The options here are `smtp`, `sendmail`, `ses`, `mailgun and postmark`.

#### SMTP & Sendmail

When using a regular SMTP provider, or sendmail, you should set the following configuration values:

- `MAIL_HOST` – This is the host for the SMTP server
- `MAIL_PORT` – This is the port that will be used to connect to the SMTP server
- `MAIL_USERNAME` – The username used to authenticate with the SMTP server
- `MAIL_PASSWORD` – The password used to authenticate with the SMTP server
- `MAIL_FROM_ADDRESS` – The address that mail will appear to come from
- `MAIL_FROM_NAME` – The name that mail will appear to come from

#### SES

When using SES as your mail service, you should set the following configuration values, adding them to the `.env` file if they are not already present:

- `AWS_ACCESS_KEY_ID` – Your AWS ID key
- `AWS_SECRET_ACCESS_KEY` – Your AWS secret key
- `AWS_DEFAULT_REGION` – Your AWS region (defaults to `us-east-1` if not included in the configuration file)

#### Mailgun

When using Mailgun as your mail service, you should set the following configuration values, adding them to the `.env` file if they are not already present:

- `MAILGUN_DOMAIN`
- `MAILGUN_SECRET`
- `MAILGUN_ENDPOINT` – (defaults to `api.mailgun.net` if not included in the configuration file)

#### Postmark

When using Postmark as your mail service, you should set the following configuration values, adding them to the `.env` file if they are not already present:

- `POSTMARK_TOKEN`