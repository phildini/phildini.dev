Title: Digitizing 55,000 pages of civic meetings
Tags: data,politics,datasette,pdfs,civics
Date: 2022-08-21

# Digitizing 55,000 pages of civic meetings

It is very hard for the average resident of a U.S. city to know what's going on with their civic government. It's even harder for them to get any sort of historical context on why things are the way they are. Let's take my hometown, the City of Alameda. Six months ago if you wanted to know which city meetings had discussed rent control, your options were:

1. Have a friend who is a constant watcher of Alameda meetings / the #alamtg hashtag and could tell you
2. Go through every meeting minutes on the [Alameda Legistar](https://alameda.legistar.com/Calendar.aspx) and hope you figured it out

This is pretty common across a lot of civic government. I don't think municipalities are willfully trying to hide this information from residents, and I don't think it's ineptitude. I think most cities, even the large ones, are understaffed, and without a concerted push it's hard to make "visibility of city documents" a priority.

It hurts and helps that many cities use Legistar for both meeting management and the complete historical archive. Legistar is only designed for one of those use cases, and it's not archiving. But the thing that Legistar does provide is an API, and one that's reasonably sane for fetching a city's meeting archives.

Because of that API, there's two tools that have come online for Alameda (and now Oakland) in the past six months that have made it much easier to answer the question: "which meetings in my city have recently talked about rent control?"

The first tool is the [Council Data Project](https://councildataproject.org/). CDP is a truly incredible effort and tool, run by some fantastic people (Hi, Eva!). I'm not going to talk much about it, other than to say you should check out the [Alameda](https://councildataproject.org/alameda/#/) and [Oakland](https://councildataproject.org/oakland/#/) instances. 

The second tool is something I've been working on recently: SQL-backed full text search of city meeting minutes. You can see this working for the cities of [Alameda](https://data.alameda.one/alameda_minutes/pages) and [Oakland](https://data.oakland.works/oakland_minutes/pages) right now. That's 18,746 pages of city minutes for Alameda, and 37,172 pages of city minutes for Oakland, now fully searchable by anyone.

So let's talk about how I did this, and how you can do this for your city, possibly with my help!

Here's my process:

1. Figure out where official city minutes are hosted
2. Write a script to fetch and format those city minutes
3. Upload all the fetched minutes to AWS S3
4. Run [s3-ocr](https://github.com/simonw/s3-ocr) across the corpus of minutes
5. Download the ocr'd pages into a sqlite DB
6. Deploy a [datasette](https://datasette.io/) instance to fly.io with that sqlite DB.
7. Post to twitter so people know about it.

Datasette and s3-ocr are wildly powerful tools written by Simon Willison that will change the landscape of preservation and archiving over the next few years. I'm focused on civic data, but these tools were _developed_ with historical records in mind, and if you haven't dug into the ecosystem yet, please do so.

So what's actually happening, step by step? Let's take Alameda as an example, because Oakland followed a fairly similar pattern. All the code (and all the PDFs) for both cities are available on Github at [phildini/alameda-data](https://github.com/phildini/alameda-data) and [phildini/oakland-data](https://github.com/phildini/oakland-data)

First we need to find all the city minutes. The [Legistar API](https://webapi.legistar.com/Help) has an Events endpoint that will give you all the city meetings in Legister. Except not quite, it'll cap at one thousand, so we need to go year-by-year and fetch all the meetings for that year. For Alameda, that means going back to 2004, for Oakland it's 2003. 

```
for year in years:
    print(f"Fetching {year}")
    event_listing_url = f"{base_api_url}events?$filter=EventDate+ge+datetime%27{year}-01-01%27+and+EventDate+lt+datetime%27{year}-12-31%27"
    list_response = requests.get(event_listing_url)
```

Once we have a list of events, we pull the date out of the Event object, parse it a little, and check to see if we've already fetched this set of minutes.

```
for event in list_response.json():
    body = event["EventBodyName"]
    directory = f"./data/{body.replace(' ','')}"
    if not os.path.exists(directory):
        os.makedirs(directory)
    filename = event["EventDate"].split("T")[0] + ".pdf"
    filepath = f"{directory}/{filename}"
    if os.path.exists(filepath):
        continue
```

If we haven't fetched it, we get the meeting listing url, then fetch the page with BeautifulSoup, grab the link to the minutes, and pull it down.

```
page_url = event["EventInSiteURL"]
page_response = requests.get(page_url)
soup = BeautifulSoup(page_response.text, "html.parser")
minutes_url = (
    base_url
    + soup.find(id="ctl00_ContentPlaceHolder1_hypMinutes").attrs["href"]
)
minutes_response = requests.get(minutes_url, allow_redirects=True)
```

With the PDF response ready, we write the file and move on.

```
with open(filepath, "wb") as minutes_pdf:
    minutes_pdf.write(minutes_response.content)
```

For Alameda, the script takes an hour or two. For Oakland, about four hours. Those times are both first-run times; because the script is idempotent, we won't re-download a file we've already downloaded. On successive runs, the script completes in minutes or seconds.

With all the PDFs downloaded and sorted, we upload to S3. Because we're using `s3-ocr --all`, each city gets its own bucket.

The OCR process takes about half an hour, but that's again a first run time. All the parts of this process are idempotent, so running s3-ocr successive times will only operate on new files. Each new file that s3-ocr discovers on a run gets a job created for it, and when all the jobs are done we run `s3-ocr index city_minutes.db`. Indexing all the pages into sqlite databases took about two hours for Alameda, and about four hours for Oakland.

Once the index process is done, we upload a datasette instance with the DBs to fly.io, a stunningly good PaaS that makes the deployment as easy as:

```
datasette publish fly city_minutes.db --app="alameda-datasette"
```

While (so far) hosting these datasets on fly.io is free, the project itself has some costs. The OCR used by s3-ocr is AWS Textract. Running s3-ocr against the 18,746 pages of Alameda city minutes cost about $40, and the 37,172 cost about $80.

With that process and pricing in mind, let's talk about Berkeley and San Francisco. The Events endpoint for the San Francisco Legistar instance is currently broken, probably because of how far back the San Francisco data goes. I have a lead on how to get around that, but it'll still cost some processinig time.

Berkeley is a fun case. Berkeley already has full-text search of it's city minutes, but that search doesn't return the text, it only returns the PDFs. Using that, I was able to get a huge chunk of Berkeley city minutes downloaded. How huge a chunk? 9700+ documents, in some cases going back to 1905. I would really love to get these digitized, and be able to present this century-long searchable record to the City of Berkeley, but my estimates are that it's going to cost $350 to run s3-ocr against the full set, and that's more than I can bear on my own right now.

This is where you, the reader, hopefully come in. If you like what I'm doing here, I'm accepting donations via [Venmo (@phildini)](https://account.venmo.com/u/phildini) and [Square Cash ($phildini)](https://cash.app/$phildini). These donations will go straight to the AWS cost of processing more city minutes data. If you know someone in Alameda or Oakland or San Francisco or _especially_ Berkeley who wants to see this information be accessible, please share this with them. If you know a political activist or journalist who could use this data, please share it with them. If you think of cool uses for this data yourself, please share them with me.

And if there's data you want exposed to the sunshine like this, for your city or others, reach out to phildini@phildini.net.
