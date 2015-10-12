When ever I am doing a admin portal there is usually a need for a humanly readable 'last seen' time in PHP. Normally there is also a lot of formatting timestamps in to readable formats. So I here are a couple of useful time PHP functions I use in my projects. 

The first is useful for formatting last seen, active since, or posted at type times into a humanly readable format. Its a little crude yes, but it gets the job done. You can find it as a [gist](https://gist.github.com/davidthingsaker/291987d961520b9b26a8) on my github and I welcome any improvements. 


```php
# Set a timestamp to time, e.g. 2 months
public static function time_ago($timestamp)
{
    $years = floor($timestamp / 31536000);
    $days = floor(($timestamp - ($years*31536000)) / 86400);
    $hours = floor(($timestamp - ($years*31536000 + $days*86400)) / 3600);
    $minutes = floor(($timestamp - ($years*31536000 + $days*86400 + $hours*3600)) / 60);
    $seconds = floor(($timestamp - ($years*31536000 + $days*86400 + $hours*3600 + $minutes*60)));
    
    $timestring = '';
    if ($years > 0){
        $timestring .= $years . ' years ';
    }
    if ($days > 0) {
        $timestring .= $days . ' days ';
    }
    if ($hours > 0) {
        $timestring .= $hours . ' hrs';
    }
    if ($minutes > 0) {
        $timestring .= $minutes . ' mins';
    }
    if ($seconds > 0) {
        $timestring .= $seconds . ' secs';
    }
    
    # Optional
    # $timestring .= ' ago';

    return $timestring;
}

```

The other function I use is named `format_timestamp`, though it can actually format a 'time string' as well. It will turn a MySQL datetime string into a humanly readable format. All you have to do is use the option `$time` parameter as false. Again you can [find this one in my gists](https://gist.github.com/davidthingsaker/291987d961520b9b26a8) and any improvements or suggestions are more than welcome. 


```php
# Formats a time stamp into a date string
public static function format_timestamp($timestamp, $time = false)
{
    if($timestamp == 0) {
        return null;
    } else if(is_string($timestamp)){
        $timestamp = strtotime($timestamp);
    }
    if ($time) {
        $date_string = Date('H:i:s jS M Y', $timestamp);
    } else {
        $date_string = Date('jS M Y', $timestamp);
    }
    return $date_string;
}
```

Check out my other gists as well, there are a few in there for formatting various things into humanly readable strings as well as other useful PHP functions. 
