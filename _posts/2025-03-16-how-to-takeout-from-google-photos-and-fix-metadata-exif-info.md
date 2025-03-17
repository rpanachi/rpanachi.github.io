---
layout: post
title: "How to backup your photos from Google (and clean up their mess)"
date: 2025-03-16 00:00:00
categories: [tutorial, google takeout, hacking, ruby, exiftool]
excerpt: How I used a Ruby script and exiftool to fix the info metadata on the Google Photos backup files
disqus: true
archive: true
---

> Any sufficiently advanced incompetence is indistinguishable from malice.<br/>
> – Grey’s Law

Downloading your whole backup from Google Photos should be easy, right? Well, not really. Google's Takeout service makes it way more annoying than it should be. Not only do you have to wait for about 3 days, manually download a bunch of ZIP files from a page with the worst usability I deal it in the last decade, and discover that the files didn't come with the info metadata on it (which keeps the correct photo date, time, location, etc), but stored separately in JSON files with inconsistent naming.


So, sharing my frustration here and how I dealt with it to clean up Google's mess and get my photos back in order.

## Step 1: Getting the backup from Google Takeout

First, I requested my export from [Google Takeout](https://takeout.google.com/). This took three days to process, since I had about 150GB of photos.

Once it was ready, I had to manually download 72 ZIP files, one by one, because Google apparently hates bulk downloads. Super fun.

![Google Takeout](/assets/images/google-takeout.png)


## Step 2: Extracting everything

After finally downloading all the files, I extracted them into a single directory with this command:

```
for zip in downloaded/*.zip; do unzip $zip -d extracted/; done
```

This dumped all the files into a structure like this:

```
extracted/
  Takeout/
    Google Fotos/
      Photos From 2024/
        image1.jpg
        image1.jpg.supplemental-metadata.json
      Photos From 2025/
        image1.jpg
        image1.jpg.supplemental-m.json
        image1.jpg.supplemental-metadata(1).json
        image1(1).jpg
        image2.jpg
        image2.jpg.sup-meta.json
```

As you can see, consistency isn't Google's strong point. I need the metadata filenames to follow the same pattern because it will be used in the next step.

## Step 3: Fixing the metadata filenames

The correct format of metadata files should be:

- `image.jpg.supplemental-metadata.json`

But I found things like:

- `image.jpg.sup-meta.json`
- `image(1).jpg.supplemental-meta.json`

To clean this up, I wrote a Ruby script that:

- Finds all metadata files with weird names.

- Renames them to match the expected format.

- If an image doesn't have a metadata file, it tries to guess the date from the filename (e.g., IMG_20240515.jpg → May 15, 2024).

Here’s the Ruby script I used:

<script src="https://gist.github.com/rpanachi/aa8a18bf090b580d6c1c2d4e9c6f51c6.js"></script>

To run it, simply execute on the terminal:

```
ruby fix_metadata.rb extracted/Takeout/Google\ Fotos/
```

The output will be like:
```
$ ruby fix_metadata.rb ./extracted/Takeout/Google\ Photos/
Total files found on ./extracted/Takeout/Google Photos/**/*: 555
Total photos from YYYY dirs found: 539
Total supported photos formats found: 131
Total metadata files found: 408

Process finalized with 3 errors:
[1/3] Unable to infer metadata for ./extracted/Takeout/Google Photos/Photos from 2013/CameraZOOM-01611c5b98.jpg
[2/3] Unable to infer metadata for ./extracted/Takeout/Google Photos/Photos from 2013/CameraZOOM-20131224200750052.jpg
[3/3] Metadata file: .//extracted/Takeout/Google Photos/Photos from 2014/IMG_1181.JPG.supplemental-metadata(1).json not exist for image: ./extracted/Takeout/Google Photos/Photos from 2014/IMG_1181(1).JPG

Process finalized with 22 fixes:
[1/22] 5142914356_01611c5b98_o.jpg.supplemental-metad.json moved to 5142914356_01611c5b98_o.jpg.supplemental-metadata.json
[2/22] 1-3922-1990-20121024123511.jpg.supplemental-me.json moved to 1-3922-1990-
...
[22/22] IMG_1182.JPG.supplemental-metadata(1).json moved to IMG_1182(1).JPG.supplemental-metadata.json

Metadata not found for 21 files:
[1/21] ./extracted/Takeout/Google Photos/Photos from 2013/CameraZOOM-20131224200623261.jpg
[2/21] ./extracted/Takeout/Google Photos/Photos from 2013/CameraZOOM-20131224200750052.jpg
...
[21/21] ./extracted/Takeout/Google Photos/Photos from 2025/IMG_5854.HEIC
```

## Step 4: Applying the metadata to files

Once the filenames were fixed, I used [exiftool](https://exiftool.org/) to write the correct metadata into the photo files:

```
exiftool -r -d %s -tagsfromfile "%d/%F.supplemental-metadata.json" \
  "-GPSAltitude<GeoDataAltitude" "-GPSLatitude<GeoDataLatitude" \
  "-GPSLatitudeRef<GeoDataLatitude" "-GPSLongitude<GeoDataLongitude" \
  "-GPSLongitudeRef<GeoDataLongitude" "-Keywords<Tags" "-Subject<Tags" \
  "-Caption-Abstract<Description" "-ImageDescription<Description" \
  "-DateTimeOriginal<PhotoTakenTimeTimestamp" \
  -ext "*" -overwrite_original -progress --ext json -ifd0:all= \
  extracted/Takeout/Google\ Fotos/
```

(just change the last argument to the directory where you extract the photos)

This does a bunch of things:
- Reads the correct timestamp from the JSON file.
- Writes it into the photo’s EXIF data.
- Restores GPS location and descriptions.
- Fixes bad tags in one go.

## Wrapping Up

Now that all the metadata is fixed, I can import everything back into iCloud Photos (my case) or any other storage service without messing up the timeline.

Let me know if you have any other tricks or improvements!

## References

- [https://legault.me/post/correctly-migrate-away-from-google-photos-to-icloud]()
- [https://www.photools.com/community/index.php?topic=13589.0]()

