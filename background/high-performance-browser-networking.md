Performance Checklist

Optimizing TCP performance pays high dividends, regardless of the type of application, for every new connection to your servers. A short list to put on the agenda:

    Upgrade server kernel to latest version.

    Ensure that cwnd size is set to 10.

    Ensure that window scaling is enabled.

    Disable slow-start after idle.

    Investigate enabling TCP Fast Open.

    Eliminate redundant data transfers.

    Compress transferred data.

    Position servers closer to the user to reduce roundtrip times.

    Reuse TCP connections whenever possible.

    Investigate "TCP Tuning for HTTP" recommendations. 