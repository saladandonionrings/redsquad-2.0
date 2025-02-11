# 🗣 Breaches Links

[haveibeenpwned.com](https://haveibeenpwned.com)

[Monitors the entire Pastebin - psbdmp.ws](https://psbdmp.ws/api/search/test@test.com)
* Usage : `https://psbdmp.ws/api/search/<email>`

[breachdirectory.org](https://breachdirectory.org/)

[Search across IP/email/domain/bitcoin address and etc. - intelx.io](https://intelx.io/)

[Snusbase.com](https://snusbase.com/)

[Almost the same as IntelX - www.dehashed.com](https://www.dehashed.com)

[Email / Username, password breaches check - leakcheck.io](https://leakcheck.io/)

[leakpeek.com](https://leakpeek.com)

[Check if the email show up in someone's stealer logs - hudsonrock.com](https://hudsonrock.com)
- Usage `https://cavalier.hudsonrock.com/api/json...@email.com`

[Find anyone that left a trace on the Internet - Free : 5 searches a day - oathnet.ru](https://oathnet.ru)

# Tools

* [WhatBreach](https://github.com/Ekultek/WhatBreach) — An OSINT tool facilitating the discovery of breaches involving a specific email address, capable of loading public databases.
* [h8mail](https://github.com/khast3x/h8mail) и [pwnedOrNot](https://github.com/thewhiteh4t/pwnedOrNot) — Tools for finding passwords from compromised email addresses in public databases.
* [Infoga](https://github.com/m4ll0k/Infoga) — Collects email account information from public sources and checks for email leaks using the haveibeenpwned.com API.

### Pastebins

[add email address in url - psbdmp.ws](https://psbdmp.ws/api/search/email/)

# Investigate - Verify Leak Data

https://techjournalism.medium.com/how-to-verify-leak-data-3b0c8d8b764a

## Metadata

- Extract the metadata from multiple images and store it in a list : [link](https://stackoverflow.com/questions/49102325/extract-metadata-from-multiple-images-and-store-it-in-a-list)
  - Tools :
    - [PDFMiner](https://pypi.org/project/pdfminer/)
    - [metadata2go.com](https://www.metadata2go.com/)
    - [Jimpl](https://jimpl.com/)
    - [VerEXIF](http://www.verexif.com/en/)
    - [Metadata Interrogator](https://www.metadataanalysis.com/)

  - Automate the collection of metadata, collect creation dates and store it for analysis in chronological order :

```python
    import fitz
    from datetime import datetime
    
    def extract_creation_date(pdf_path):
        with fitz.open(pdf_path) as doc:
            try:
                creation_date = doc.metadata.get("creationDate")
                if creation_date:
                    creation_date = datetime.strptime(creation_date[2:16], "%Y%m%d%H%M%S")
                    return creation_date
                else:
                    return None
            except Exception as e:
                print(f"Error extracting creation date from {pdf_path}: {e}")
                return None
    
    if __name__ == "__main__":
        pdf_paths = ["file1.pdf", "file2.pdf", "file3.pdf"]
        creation_dates = []
        for path in pdf_paths:
            creation_date = extract_creation_date(path)
            if creation_date:
                creation_dates.append((path, creation_date))
        
        creation_dates.sort(key=lambda x: x[1])
        print("PDF Creation Dates (in chronological order):")
        for i, (pdf_path, date) in enumerate(creation_dates, start=1):
            print(f"{i}. {pdf_path} - {date}")
```

## Virus scan - You never know

- Set up a separate machine, one that perhaps connects via the [Tor Project](https://www.torproject.org/download/) 
  - Runs two different virus scan software packages. 
    - If the data runs on an external hard drive, check virus/malware on that dump of data as a whole.

## Check photos/graphs

- [Yandex reverse image search](https://www.skopenow.com/resource-center/image-based-osint-investigations-tips-techniques) : Struggles with graphics
- Search through Dorks, a lot of leaks on [Slideshare](https://de.slideshare.net/vladdroz/russiathe-way-from-electronic-to-open-government)
- [Chronolocate](https://www.bellingcat.com/tag/chronolocation/)

## Signatures (pen)

- [fileproinfo](https://fileproinfo.com/tools/comparison/signature)

## Word-Doc

- [FOCA](https://github.com/ElevenPaths/FOCA)
