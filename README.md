AP-date-style
=============

Create a date variable for Drupal 7 that adheres to the AP style

## How coding languages sees months
Coding languages generally have three different human-readable  ways to display months:

1. spell it out; January is January, March is March, etc.

2. abbreviate with a set number of characters; January is Jan., March is Mar., etc.

3. use numbers; Janury is 1, March is 3, etc.

See the php date format manual: http://us3.php.net/manual/en/function.date.php

## How AP style thinks you should write months
From the 2012 AP stylebook:
>**months** Capitalize the names of months in all uses. When a month is used with a specific date, abbreviate only *Jan., Feb., Aug., Sept., Oct., Nov.,* and *Dec.* Spell out when using alone, or with a year alone.

>When a phrase lists only a month and year, do not separate the year with commas, When a phrase refers to a month, day and year, set off the year with commas.

>EXAMPLES: *January 1972 was a cold month. Jan. 2 was the coldest day of the month. His birhday is May 8. Feb. 14, 1987, was the target date. She testified that it was Friday, Dec. 3, when the accident occurred.*

>In tabular material, use these three-letter forms without a period: *Jan, Feb, Mar, Apr, May, Jun, Jul, Aug, Sep, Oct, Nov, Dec.*

## The problem
Adding dates to content in Drupal is very useful and often very necessary. The best way to add a date to a piece of content is to add a date field to the content type. Adding a date field instead of a simple text field allows the system to sort and organize your content in specific ways.

When we go to format our date fields in example.com/admin/config/regional/date-time/formats, we see that we may only use one of the three coding styles outlines above.

So how do we acheive our AP style date?

## The solution
We need to create a variable in our template.php that sees our date field, runs it through an if statement and outputs our AP styled date format.

## The process
- **Add a date field to your content type** You can choose what type of date field depending on your specific needs. Here is a guide to the pros/cons of each date type: https://drupal.org/node/1326872
- **Add code to template.php** Add this code to your theme's template.php file under the `function yourTHEME_preprocess_node(&$variables, $hook){` for the variable to apply to all content types or `function yourTHEME_preprocess_node_contentTYPE(&$variables, $hook){` for the variable to apply just to a choosen content type.

The code:

    //AP date style  
    $post_date_raw = new DateTime($variables['node']->field_original_post_date['und'][0]['value']);  
    $post_date_month = $post_date_raw->format('F');  
    $post_date_day_year = $post_date_raw->format('j, Y');  
  
    if($post_date_month == "September"){  
    	$ap_date_sept = "Sept. " . $post_date_day_year;  
        $variables['ap_date'] = $ap_date_sept;  
    }elseif(strlen($post_date_month) > 5){  
        $ap_date_short = substr($post_date_month, 0,3) . "." . " " . $post_date_day_year;  
        $variables['ap_date'] = $ap_date_short;  
    }else{  
        $ap_date_long = $post_date_month . " " . $post_date_day_year;  
        $variables['ap_date'] = $ap_date_long;  
    }
- **Add code to node--contentTYPE.tpl.php file** Add the variable we created to your node template file
`<?php print $ap_date; ?>`

## The code explained

    //AP date style  
    $post_date_raw = new DateTime($variables['node']->field_original_post_date['und'][0]['value']);

Create a variable called `$post_date_raw` to house our new DateTime object. We create the object from the field we added to our content type. I dug into the field elements with the devel module to discover the path of the variable: `$variables['node']->field_original_post_date['und'][0]['value']`. 

`field_original_post_date` should be replaced with your variable name.

    $post_date_month = $post_date_raw->format('F');
    $post_date_day_year = $post_date_raw->format('j, Y');

Create variables to print the month and the day and year separately. F prints the full month name, January is January, March is March, etc. j, Y prints the day and year in this format: 23, 2014.

    if($post_date_month == "September"){  
        $ap_date_sept = "Sept. " . $post_date_day_year;  
        $variables['ap_date'] = $ap_date_sept; 

We want to start our if statement with the oddball, September. With longer months, we usually would want to shorten it to three letters and a period. However, with September, we need to shorten the month name to four letters and a period. So, if the month equals September, set the `$ap_date_sept` variable to Sept. plus our day and year.

Since Drupal has a specific way it wants variables set,  we use `$variables['ap_date']` to equal the `$ap_date_sept` variable we just created. This way we can use `$ap_date` in our node--contentTYPE.tpl.php file.

    }elseif(strlen($post_date_month) > 5){  
        $ap_date_short = substr($post_date_month, 0,3) . "." . " " . $post_date_day_year;  
        $variables['ap_date'] = $ap_date_short;  

The next part of our if statement checks for longer months. If the month is longer than five characters, then we want to shorten it to three characters using the php string function `substr()`. Then we add a period and the day and year. 

Again, because Drupal has a specific way it wants variables set, we use `variables['ap_date']` to equal the `$ap_date_short` variable we just created.

    }else{  
        $ap_date_long = $post_date_month . " " . $post_date_day_year;  
        $variables['ap_date'] = $ap_date_long;  
    }

The last part of our if statement covers months under five characters, months that we don't want to alter the disply of. If the month is shorter than five characters, simply print it and add the day and year at the end.

Make sure the Druapl variable equals the variable we just set up (`$ap_date_long`) in case the condition is true.

And there we have it.