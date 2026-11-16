# Vultr speed test: how do you test all 33 locations, which file should you pull, and why do one-line scripts report 7 KB/s? (Commands, endpoint list, reading the numbers)

A Vultr speed test takes two commands. Ping the `*-ping.vultr.com` host for the location you're considering, then download `vultr.com.100MB.bin` from that same host. Vultr's own FAQ publishes 33 locations, each with a Looking Glass host and two test files, so you can measure the route from your connection before you pay anything.

What follows is the full endpoint list, the commands that work, and how to read the results. Most of the guides ranking for this query hand you a one-line script and a table of millisecond values, and both are more misleading than they look.

**The short version for anyone deciding now:** test first, order in the location that returns the lowest average ping from *your* network, and start on the smallest Cloud Compute instance. The current entry tier is `vc2-1c-0.5gb-v6` at $0.0034/hr, billed at roughly $2.48/mo according to a third-party Vultr price tracker updated 27 October 2026. New accounts can also enter code **FLY300VULTR** at checkout for $300 in promotional credit; Vultr's own coupons page states it's for new customers only, covers select products, and can't be combined with other offers.

👉 [Check the current plan list and hourly rates on Vultr](https://www.vultr.com/pricing/?ref=9738262-9J)

## What a Vultr plan actually costs before you start testing

Testing is free. Ordering isn't, so it helps to know the shape of the price list before you pick a location.

Vultr doesn't sell one product with three tiers. The Cloud Compute catalogue alone runs to 122 pricing tiers, from a half-gigabyte instance up to 96-vCPU machines, and every one of them is billed hourly with a monthly cap. Prices differ by region, so the number you see on a checkout page for Frankfurt won't always match the one for New Jersey.

| Instance family | Example configuration | Listed price | Billing |
| --- | --- | --- | --- |
| Cloud Compute, Regular (vc2) | 1 vCPU / 0.5 GB RAM | $0.0034/hr · $2.48/mo | Hourly, capped monthly |
| Cloud Compute, High Frequency | NVMe storage, higher clock speed | Varies by region | Hourly, capped monthly |
| Cloud Compute, High Performance | AMD or Intel, dedicated CPU share | Varies by region | Hourly, capped monthly |
| Optimized Cloud Compute | `voc-c-16c-32gb-500s-amd`: 16 vCPU / 32 GB | $360.04/mo for this SKU; family entry around $28/mo | Hourly, capped monthly |
| Cloud GPU (vcg) | `vcg-a16-6c-60g-16vram`: 6 vCPU / 60 GB / 16 GB VRAM | $0.4712/hr · $343.98/mo | Hourly, capped monthly |
| High-memory flagship | `vx1-g-96c-384g`: 96 vCPU / 384 GB | $2,102.40/mo | Hourly, capped monthly |
| Bare Metal | Whole dedicated server | From $120/mo in G2's summary | Monthly |
| Add-ons | Managed Databases / DDoS protection | From $15/mo / $10/mo | Monthly |

Prices above come from a third-party price tracker (snapshot dated 27 October 2026, quoted for one region) and G2's pricing summary, not from a live official page fetch, so treat them as the shape of the catalogue rather than a quote. Hourly billing is the part that matters for testing: you can spin up a machine in a candidate location, run your own throughput checks from inside it, and destroy it the same day.

If your reason for reading this is "which location should I buy in", the answer comes from the ping test, not from the price table. The cheapest plan costs the same in every region.

## The official test endpoints: 33 locations, two files each

Vultr's FAQ answers "How can I test Vultr download speeds?" with a table of Looking Glass hosts and download files. Every location follows one naming rule, which saves you from memorising 66 URLs:


https://<looking-glass-host>/vultr.com.100MB.bin
https://<looking-glass-host>/vultr.com.1000MB.bin


For Silicon Valley, that's `https://sjo-ca-us-ping.vultr.com/vultr.com.100MB.bin`. The Looking Glass host itself also responds to ping and opens in a browser as a network information page.

**Americas**

| Location | Looking Glass host |
| --- | --- |
| Toronto, Canada | `tor-ca-ping.vultr.com` |
| New York (NJ) | `nj-us-ping.vultr.com` |
| Chicago, Illinois | `il-us-ping.vultr.com` |
| Atlanta, Georgia | `ga-us-ping.vultr.com` |
| Dallas, Texas | `tx-us-ping.vultr.com` |
| Miami, Florida | `fl-us-ping.vultr.com` |
| Mexico City, Mexico | `mex-mx-ping.vultr.com` |
| Seattle, Washington | `wa-us-ping.vultr.com` |
| Los Angeles, California | `lax-ca-us-ping.vultr.com` |
| Silicon Valley, California | `sjo-ca-us-ping.vultr.com` |
| São Paulo, Brazil | `sao-br-ping.vultr.com` |
| Santiago, Chile | `scl-cl-ping.vultr.com` |
| Honolulu, Hawaii | `hon-hi-us-ping.vultr.com` |

**Europe**

| Location | Looking Glass host |
| --- | --- |
| Manchester, England | `man-uk-ping.vultr.com` |
| London, UK | `lon-gb-ping.vultr.com` |
| Amsterdam, NL | `ams-nl-ping.vultr.com` |
| Madrid, Spain | `mad-es-ping.vultr.com` |
| Paris, France | `par-fr-ping.vultr.com` |
| Frankfurt, DE | `fra-de-ping.vultr.com` |
| Stockholm, Sweden | `sto-se-ping.vultr.com` |
| Milan, Italy | `mxp-it-ping.vultr.com` |
| Warsaw, Poland | `waw-pl-ping.vultr.com` |

**Asia-Pacific, Middle East and Africa**

| Location | Looking Glass host |
| --- | --- |
| Tel Aviv, Israel | `tlv-il-ping.vultr.com` |
| Tokyo, Japan | `hnd-jp-ping.vultr.com` |
| Osaka, Japan | `osk-jp-ping.vultr.com` |
| Seoul, South Korea | `sel-kor-ping.vultr.com` |
| Delhi NCR, India | `del-in-ping.vultr.com` |
| Mumbai, India | `bom-in-ping.vultr.com` |
| Bangalore, India | `blr-in-ping.vultr.com` |
| Singapore | `sgp-ping.vultr.com` |
| Sydney, Australia | `syd-au-ping.vultr.com` |
| Melbourne, Australia | `mel-au-ping.vultr.com` |
| Johannesburg, South Africa | `jnb-za-ping.vultr.com` |

Vultr also keeps a documentation search endpoint at `vultr.com/docs/search/?query=speed`, worth a look if a host above ever stops responding.

## Running the test: latency first, throughput second

Latency tells you whether a location is plausible. Throughput tells you whether the route can carry your traffic. Run them in that order.

**Step 1 — latency.** Four packets is enough to spot a bad route, and more than enough to fool you if you only run it once.

bash
ping -c 4 sjo-ca-us-ping.vultr.com


Read the `avg` figure and the `stddev`. A 170 ms average with 0.9 ms deviation is a stable route. A 300 ms average with 34 ms deviation means the path is congested or taking a weird detour, and your users will feel it as stutter rather than slowness.

**Step 2 — download from the same host.** `curl` writes the transfer rate for you:

bash
curl -o /dev/null -w 'avg %{speed_download} B/s over %{time_total}s\n' \
  https://sjo-ca-us-ping.vultr.com/vultr.com.100MB.bin


`speed_download` is bytes per second. Multiply by 8 for bits, divide by 1,000,000 for MB/s. A 100MB file on a 100 Mbps line finishes in about 8 seconds, so a single file gives you a rough ceiling fast.

**Which file should you use?** The 100MB file is for shortlisting five or six locations in a couple of minutes. Switch to the 1GB file once you've narrowed it down, because a short transfer mostly measures TCP ramping up. The 1GB run takes long enough to show whether the rate holds, drops, or collapses partway through, and that sustained number is the one worth comparing against the plan you're about to buy.

**Step 3 — sweep every location.** Paste this and walk away; it pings all 33 hosts in sequence.

bash
hosts="tor-ca-ping nj-us-ping il-us-ping ga-us-ping tx-us-ping fl-us-ping \
mex-mx-ping wa-us-ping lax-ca-us-ping sjo-ca-us-ping man-uk-ping lon-gb-ping \
ams-nl-ping mad-es-ping par-fr-ping fra-de-ping sto-se-ping mxp-it-ping \
waw-pl-ping sao-br-ping scl-cl-ping tlv-il-ping hnd-jp-ping osk-jp-ping \
sel-kor-ping hon-hi-us-ping del-in-ping bom-in-ping jnb-za-ping blr-in-ping \
sgp-ping syd-au-ping mel-au-ping"

for h in $hosts; do
  printf '%-22s ' "$h"
  ping -c 4 -q "$h.vultr.com" | tail -1
done


Sort the output by average and the geography usually sorts itself.

## The one-line script everyone shares, and what it actually measures

There's a popular GitHub script, `nextkitt/vultr-speed-test`, that runs ping and download across a list of Vultr hosts from a single `curl | bash` command. It works, and it will finish in under a minute. Three things about it are worth knowing before you trust the output.

**It covers 15 locations, not 33.** The script's internal list holds Tokyo, Singapore, Sydney, Frankfurt, Amsterdam, London, Paris, Seattle, Silicon Valley, Los Angeles, Chicago, Dallas, New York (NJ), Atlanta and Miami. Missing: Toronto, Mexico City, Manchester, Madrid, Stockholm, Milan, Warsaw, São Paulo, Santiago, Tel Aviv, Osaka, Seoul, Honolulu, all three Indian locations, Johannesburg and Melbourne. If the location you're considering isn't in that list, the script tells you nothing about it.

**Each host gets four pings, fired in parallel.** That's fine for a rough ordering and useless for a fine one.

**The download test is killed after six seconds.** The script starts every download at once, then runs `sleep 6` followed by a `ps | grep | xargs kill` on all the curl processes. The progress lines in its published output show it: `Total 100M`, `Received` between 15 KB and 38 KB, `Time 0:00:05`. Those reported rates, 7,297 B/s at the top and 761 B/s at the bottom, are the average over the first few seconds of a truncated transfer, during connection setup. They are not the bandwidth of any Vultr location, and a reading of "Vultr gives me 7 KB/s" is simply a misreading of a killed download.

Two smaller details: the script requests over plain `http://` rather than HTTPS, and it has no IPv6 test, no upload test and no traceroute. Its Chinese region labels are also wrong in places (Seattle and Chicago are tagged as US East), so go by the city names in the official table.

The reason the script is still useful is speed of triage. Run it, take the three or four hosts with the lowest ping, then test those properly with the commands above using the 1GB file.

## Reading the results without fooling yourself

The same repository contains two ping reports from different runs. In the README, Tokyo averages 213.5 ms and New Jersey 217.4 ms. In the script's own comments, Tokyo is 219.1 ms and New Jersey 218.0 ms. The order flips between two runs of identical code from the same machine.

That's the whole lesson about single samples. At the 200 ms mark, four packets can't separate two locations that are within a few milliseconds of each other. If you're choosing between two candidates that close, run each three times, at different hours, and compare the medians.

For scale, here are the numbers from that author's published output, from one vantage point on one day: US West (Silicon Valley, Los Angeles) at roughly 173–179 ms, US East and central in the 200–240 ms band, Tokyo/Singapore/Sydney between 213 and 273 ms, and Europe from 249 ms (Paris) to 322 ms (London, with a 381 ms maximum and 34 ms deviation). Those figures describe the route from that person's network to each city. They are not a ranking of Vultr data centres, and your numbers will be different, possibly by a lot.

One more thing the table can't fix: distance. If you're in Singapore and the nearest Vultr region is Singapore, you'll get single-digit to low-double-digit milliseconds, and no plan upgrade improves that. If your users are in Europe and you bought in California, 150 ms of the delay is the Pacific Ocean, and the fix is a second deployment, not a bigger instance.

👉 [Start with the smallest Cloud Compute instance in the location your test picked](https://www.vultr.com/pricing/?ref=9738262-9J)

## Official Looking Glass vs third-party speed tests

These answer different questions, and people conflate them constantly.

A Looking Glass test measures the path between your connection and one Vultr facility. It's the right tool for choosing a region, because it isolates the variable you control. It is also, by design, a best-case measurement: a clean route to a host that exists purely to be measured.

Third-party test sites argue the opposite case. TestMy.net, in a page updated 7 November 2026, states that providers running their own speed tests is a conflict of interest and that "a third party opinion is necessary", since self-hosted tests eliminate the internet route from the measurement. Its pitch is that you want to know how fast your connection is to the places you actually visit, not how fast it is inside a provider's network.

Both positions are defensible because they measure different legs. Use the Looking Glass when you're deciding where to deploy. Use a third-party test when you're diagnosing why a site on your server feels slow for visitors in another country, since that path involves transit networks Vultr doesn't control and your Looking Glass never touches. Neither one settles "who is faster", and no published comparison of the two for Vultr exists in the sources behind this article.

## Using a spare VPS as a control point

If you suspect your local ISP is the problem, put a machine on the other side of the measurement. PIA's blog described this approach in 2018: create a Vultr VPS in the region closest to you, run an open-source throughput tester from it, and compare the result against what you get at home. A server in a data centre measures the route without your home connection's congestion, Wi-Fi or traffic shaping in the way.

Treat the how-to as historical. That article assumes Debian 9, which is long past end of life, depends on a build script that compiled curl from source, and points at a testing project that had already stopped in May 2017. Its "$0.01 an hour" figure is eight years stale. The method still works: rent a small instance hourly, measure from inside it, destroy it. The specific commands don't.

## Questions that come up while testing

**What counts as a good ping to a Vultr location?** Under 30 ms is local, 30–80 ms is comfortable for interactive work like SSH and databases, 150–250 ms is workable for websites served to humans, and anything above 300 ms with high deviation will feel sluggish for anything synchronous.

**Why do the 100MB and 1GB files give me different numbers?** Short transfers spend much of their time in TCP slow start, so they tend to under-report. Longer transfers average that ramp away and expose whether the rate is sustained. If the 1GB test ends far below the 100MB result, something is throttling or congesting mid-transfer.

**Why does my site still feel slow when the speed test looks fine?** The test measures one file transfer between two points. A page load is dozens of requests, TLS handshakes, database queries and whatever your plan's CPU can push. A $2.48 instance serving a heavy CMS will be CPU-bound long before the network is.

**Should I move regions for a lower ping?** Only if the difference is large and your users are near the faster location. Moving from 220 ms to 175 ms for a single admin in one country is rarely worth a rebuild; adding a second small instance in the region your visitors actually use usually is.

**Does the test traffic count against my plan?** The test files are served from Vultr's speed-test infrastructure, not from your instance, so pulling them is a test of the route rather than a charge against an instance you've deployed.

## How to decide, and two things to check before you pay

Simple sites, cron jobs, a personal VPN, a staging box: run the ping sweep, pick the lowest average with a sane deviation, and order the entry Cloud Compute tier. You can resize later without redeploying.

Anything that moves real data, streaming, backups, large uploads: shortlist three locations with the 100MB file, then confirm with the 1GB file and watch whether the rate holds to the end. Pick the location that sustains, not the one that starts fastest.

Users spread across continents: stop optimising a single ping. Two small instances in two regions cost less than one large instance and fix a problem no plan upgrade can.

Two things to confirm at checkout. First, the promotional credit: Vultr's coupons page lists **FLY300VULTR** ($300), **FLY200** ($200), **VULTRMATCH** (deposit match up to $100) and **250VULTRFLY** ($250), all new-customer only, all restricted to select products and all non-stackable, so pick one and read the terms attached to it. Second, billing: everything is hourly with a monthly cap, which means the price you see per month is what a full month of running costs, not a minimum commitment.

👉 [Compare current Vultr plans and apply a new-customer code](https://www.vultr.com/pricing/?ref=9738262-9J)
