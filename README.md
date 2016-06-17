[![Build Status](https://travis-ci.org/crabstudio/cakephp-scheduler.svg?branch=master)](https://travis-ci.org/crabstudio/cakephp-scheduler) [![Latest Stable Version](https://poser.pugx.org/crabstudio/scheduler/v/stable)](https://packagist.org/packages/crabstudio/scheduler) [![Total Downloads](https://poser.pugx.org/crabstudio/scheduler/downloads)](https://packagist.org/packages/crabstudio/scheduler) [![Latest Unstable Version](https://poser.pugx.org/crabstudio/scheduler/v/unstable)](https://packagist.org/packages/crabstudio/scheduler) [![License](https://poser.pugx.org/crabstudio/scheduler/license)](https://packagist.org/packages/crabstudio/scheduler)

# CakePHP 3: Scheduler Plugin
========================

Makes scheduling tasks in CakePHP much simpler.

Author
------
Trent Richardson [http://trentrichardson.com]

Contributor
------
Anh Tuan Nguyen [anhtuank7c@hotmail.com]

License
-------
Copyright 2015 Trent Richardson

You may use this project under MIT license.
http://trentrichardson.com/Impromptu/MIT-LICENSE.txt

How It Works
------------
SchedulerShell works by scheduling one cron (SchedulerShell) for your project. Then in bootstrap.php you can create intervals for all your tasks.  Deploying new scheduled tasks are now much easier; one crontab entry executes all your tasks.

Install
-------
You can install this plugin into your CakePHP application using [composer](http://getcomposer.org).

The recommended way to install composer packages is:

```
composer require crabstudio/scheduler
```
Or add the following lines to your application's **composer.json**:

```
"require": {
    "crabstudio/scheduler": "^1.0"
}
```
followed by the command:

```
composer update
```

## Load plugin
Open `terminal/command line` then enter following command:

```
bin/cake plugin load Scheduler
```

Or add this line to **Your_project\config\bootstrap.php**
```
Plugin::load('Scheduler');
```

Schedule a single system cron by the shortest interval you need for SchedulerShell.php.  For example, if you have 5 tasks and the most often run is every 5 minutes, then schedule this cron to run at least every 5 minutes. For more help see [Shells as Cron Jobs](http://book.cakephp.org/3.0/en/console-and-shells/cron-jobs.html).

Example cron job:

````
*/5 * * * * cd /path/to/app && bin/cake Scheduler.Scheduler
````

This would run the SchedulerShell every 5 minutes.

Now once this shell is scheduled we are able to add our entries to bootstrap.php.  Lets say we want to schedule a CleanUp task daily at 5am and a NewsletterTask for every 15 minutes.

```php
Configure::write('SchedulerShell.jobs', array(
	'task_01' => array('interval' => 'next day 5:00', 'task' => 'EmailQueue'),// tomorrow at 5am
	'task_02' => array('interval' => 'PT15M', 'task' => 'Newsletter') //every 15 minutes
));

```

## [DateInterval](http://www.php.net/manual/en/class.dateinterval.php)

    `PT1S` task will run each 1 Second
    `PT1M` task will run each 1 Minute
    `PT1H` task will run each 1 Hour
    `P1D` task will run each 1 Day
    `P1W` task will run each 1 Week
    `P1M` task will run each 1 Month
    `P1Y` task will run each 1 Year
## [datetime.formats.relative](http://php.net/manual/en/datetime.formats.relative.php)

    `next day 05:00` task will on tomorrow at 05:00
    `sunday 05:00` task will on sunday at 05:00
    `weekday 05:00` task will on from Mon-Friday at 05:00
    `saturday 05:00` task will on Saturday at 05:00
    `sun 05:00` task will on Sun at 05:00
    `sun 05:00 next month` task will on Sun at 05:00 in next month
    `Monday next week 2020-01-01` task will on Monday in next week 2020-01-01

The key to each entry will be used to store the previous run.  *These must be unique*!

**interval** is set one of two ways.
1) For set times of day we use PHP's [relative time formats](http://www.php.net/manual/en/datetime.formats.relative.php): "next day 5:00".

2) To use an interval to achieve "every 15 minutes" we use [DateInterval](http://www.php.net/manual/en/class.dateinterval.php) string format: "PT15M".

**task** is simply the name of the Task.

There are a couple optional arguments you may pass: "action" and "pass".

**action** defaults to "execute", which is the method name to call in the task you specify.

**pass** defaults to array(), which is the array of arguments to pass to your "action".

```php
Configure::write('SchedulerShell.jobs', array(
	'CleanUp' => array('interval' => 'next day 5:00', 'task' => 'CleanUp', 'action' => 'execute', 'pass' => array()),
	'Newsletters' => array('interval' => 'PT15M', 'task' => 'Newsletter', 'action' => 'execute', 'pass' => array())
));
```

Storage of Results
------------------
SchedulerShell keeps track of each run in a json file.  By default this is stored in TMP and is named "cron_scheduler.json".

If you need to change either of these you may use:

```php
// change the file name
Configure::write('SchedulerShell.storeFile', "scheduler_results.json");
// change the path (note the ending /)
Configure::write('SchedulerShell.storePath', "/path/to/save/");
```

Preventing Simultaneous SchedulerShells Running Same Tasks
----------------------------------------------------------
By default, the SchedulerShell will exit if it is already running and has been for less than 10 minutes. You can adjust this by setting:

```php
// change the number of seconds to wait before running a parallel SchedulerShell; 0 = do not exit
Configure::write('SchedulerShell.processTimeout', 5*60);
```

Other Notes/Known Issues
------------------------
- The optional pass arguments have not been thoroughly tested
- PHP prior to version 5.3.6 only used relative datetime for the DateTime::modify() function. This could result in an interval of "next day 5:00" not running if the previous `lastRun` time was 05:02. Therefore this plugin should only be run on PHP >= 5.3.6.
