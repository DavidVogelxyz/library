# Downloading video files using ffmpeg

While `yt-dlp` is the go-to program on Linux for downloading YouTube video and audio, sometimes video files aren't hosted on YouTube (or a similar video platform).

To download a video file that's been uploaded to a different platform, one can use `ffmpeg`. `ffmpeg` is a command line tool that allows the user to manipulate video content, but it can also be used to take in an input and direct the output to a MP4 file.

An example of a legitimate use case for this command was a scenario at work where a team member recorded an instructional video through Microsoft Teams. While the recording was posted to the company's Microsoft SharePoint, other team members were unable to download the video and access it offline. In addition, an expiration date and time was forced onto the video, which meant that the company and team members would lose access to the tutorial within a few months. By downloading the content and storing it on local drives, the tutorial video was able to be preserved and shown to other team members in the future.

To download a video file using `ffmpeg`, use the following command:

```
ffmpeg -i "$URL" -c copy $FILE_NAME.mp4
```

The options used are the following:

- `-i` specifies the input; in this case, the URL linking to the raw video content.
- `-c` specifies the codec to be used; in this case, the codec "copy" tells `ffmpeg` that the input streamed should *not* be re-encoded.

However, the important note is that the "$URL" is not the URL to the page where the video can be found -- it is instead the URL *for* the video.

This can be accessed by going into the browser's "developer tools" ("F12" on most browsers), going to the "network" tab, and grabbing the video's actual URL. This will often be a "fetch", "video", or "media" type object (sometimes, "initiator") with a name starting with "videoplayback" or "videomanifest." Right click this object and copy the link address to that object, and substitute that for $URL in the command (within the quotation marks).

Some notes on using the browser's developer tools to find the correct object and URL:

- The object almost always has "videomanifest" in the name.
- The type is often "fetch"; but, it can also be "dash".

In a situation where the audio and video have been split into separate files, the option `-i` can be used multiple times to specific both the audio file and the video file to be copied into the final output file. This can be seen in the example below:

```
ffmpeg -i "$video_URL" -i "$audio_URL" -c copy $FILE_NAME.mp4
```

## References

- [Reddit - Is there any way to download view only videos from sharepoint?](https://www.reddit.com/r/sharepoint/comments/nuk8q0/is_there_any_way_to_download_view_only_videos/)
    - The original Reddit post where the solution was found.
- [Linux | Security | Networking - How to Download View-Only Teams Meeting Recording Video from SharePoint](https://www.lisenet.com/2022/how-to-download-view-only-teams-meeting-recording-video-from-sharepoint/)
    - According to the Reddit users with the solution, this was the source.
- [Imgur - Image of the correct object in the browser's developer tools](https://imgur.com/a/V0IfOxM)
