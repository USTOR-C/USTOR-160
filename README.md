# USTOR-160 - A Linux/C solution for Tracking UID's and Unique Capping.

### Initially intended for use with Advertising Servers or Real-Time Bidding (OpenRTB) Demand-Side Platforms (DSP).

*Term: Identifier for advertisers (IDFA) - a unique user id*

This will allow accurate and high performance for low cost single-server RTB
and DSP solutions, there is no set time for IDFA expirary as it is automatic,
based on your average daily win rate, the buffers MAX_SITES and MAX_IDFA_PER_SITE
will fill accordingly. The idea here is that you block X amount of users for as
long as you possibly can with the quantity of memory allocated.

The idea here is if your winning around one million impressions a day, from roughly
250 different publisher/site id's then MAX_SITES 400 and MAX_IDFA_PER_SITE 4000
would seem a suitable memory requirement to ensure IDFA's where kept for around
24 hours.

The buffer write is cyclic/ring buffer so oldest IDFA's are the first to be forgotten
when the buffer does finally fill and loop back to it's beginning.
This creates am effect where smaller sites sending less traffic have their IDFA's
blocked for a longer period of time. Minimizing repeat traffic from lower supplying
traffic sources.

You will get notifications in the console which let you know when site buffers max
and trigger a loop, all are timestamped.

Execute the USTOR process and you will be able to use the php functions below
on the same local server.

USTOR opens a TCP port 6810 and then allows you to send two simple commands
as can be seen in the php code below, which allow adding an IDFA and checking
to see if an IDFA has been added.

The SITES / IDFA bucketing keeps array iterations low(er), and the ring buffer write
head with all the memory pre-allocated keeps writes O(1) while still being
advantageous to culling oldest IDFA's first.

For small single server solutions, a local instance of USTOR-160 should have more
than adequate performance and reliability.

PHP Functions for Write and Check operations

```
function add_ustor($source, $idfa)
{
    $fp = stream_socket_client("tcp://127.0.0.1:6810", $errno, $errstr, 1);
    if($fp)
    {
        fwrite($fp, "$" . sha1($source) . " " . sha1($idfa));
        fclose($fp);
    }
}
```

```
function check_ustor($source, $idfa)
{
    $fp = stream_socket_client("tcp://127.0.0.1:6810", $errno, $errstr, 1);
    if($fp)
    {
        $r = fwrite($fp, sha1($source) . " " . sha1($idfa));
        if($r == FALSE)
        {
            fclose($fp);
            return TRUE;
        }
        $r = fread($fp, 1);
        if($r != FALSE && $r[0] == 'y')
        {
            fclose($fp);
            return TRUE;
        }
        fclose($fp);
        return FALSE; //FALSE = It's not stored, you can bid :)
    }
    return TRUE;
}
```
