# MD scrub

## Motivation

Quoting the "Scrubbing and mismatches" section of the md(4) man page:

> As storage devices can develop bad blocks at any time it is valuable to
> regularly read all blocks on all devices in an array so as to catch such bad
> blocks early. This process is called scrubbing.
>
> md arrays can be scrubbed by writing either check or repair to the file
> md/sync_action in the sysfs directory for the device.
>
> Requesting a scrub will cause md to read every block on every device in the
> array, and check that the data is consistent. For RAID1 and RAID10, this
> means checking that the copies are identical. For RAID4, RAID5, RAID6 this
> means checking that the parity block is (or blocks are) correct.
>
> If a read error is detected during this process, the normal read-error
> handling causes correct data to be found from other devices and to be written
> back to the faulty device. In many case this will effectively fix the bad
> block.
>
> If all blocks read successfully but are found to not be consistent, then this
> is regarded as a mismatch.

**NB:** As explained by the `md`(4) man page, *a mismatch does not necessarily
mean that the RAID array is defect* - it can also occur during normal operation
in some use cases.

## Usage

To run regular *check* scrubs of a RAID array (e.g., *md0*), run

    systemctl enable --now md-check@md0.timer

You can enable scrubbing of multiple arrays in this way.

By default, scrubbing is done twice a year (on average). Use the standard
systemd override mechanisms to customize the schedule.

Alternatively, define cron job(s) that invoke `mdscrub` directly at the exact
time(s) you want.

## Implementation

The `mdscrub` script starts a scrub on a RAID array, and waits for it
to finish, then reports the number of mismatches found. The script is primarily
intended for use by the systemd `md-check@.service` unit, but it can also be
invoked directly.

The `md-check@.timer` unit is for triggering scrubbing of an array at regular
intervals - by default, semi-annually. Note that it adds a random delay of up
to 2 months to reduce the chance of multiple arrays being scrubbed
concurrently.

## References

 1. [md(4), section "Scrubbing and Mismatches"](https://linux.die.net/man/4/md)
 1. [Data scrubbing (Wikipedia)](https://en.wikipedia.org/wiki/Data_scrubbing)
 1. [Linux RAID sync operations](https://raid.wiki.kernel.org/index.php/RAID_Administration)
 1. [Linux RAID](https://raid.wiki.kernel.org/)


