# Low-Latency DASH and HLS streaming

OvenMediaEngine supports HTTP based streaming protocols such as HLS, MPEG-DASH\(Hereafter DASH\) and Low-Latency MPEG-DASH with CMAF\(Hereafter LLDASH\).  

<table>
  <thead>
    <tr>
      <th style="text-align:left">Title</th>
      <th style="text-align:left">Functions</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Delivery</td>
      <td style="text-align:left">HTTP for HLS and DASH
        <br />HTTP 1.1 chunked transfer for LLDash</td>
    </tr>
    <tr>
      <td style="text-align:left">Security</td>
      <td style="text-align:left">TLS (HTTPS)</td>
    </tr>
    <tr>
      <td style="text-align:left">Format</td>
      <td style="text-align:left">
        <p>TS for HLS</p>
        <p>ISOBMFF for DASH</p>
        <p>CMAF for LLDASH</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Codec</td>
      <td style="text-align:left">H.264, AAC</td>
    </tr>
  </tbody>
</table>

{% hint style="info" %}
OvenMediaEngine will support Low-Latency HLS in the near future. We are monitoring an  Apple's situation.  
{% endhint %}

## Configuration

To use HLS, Dash and LLDash, you need to add the `<HLS>` and`<DASH>` elements to the `<Publishers>` in the configuration as shown in the following example.

```markup
<Server version="5">
    <Bind>
        <Publishers>
            <HLS>
                <Port>80</Port>
            </HLS>
            <DASH>
                <Port>80</Port>
            </DASH>
        </Publishers>
    </Bind>
    <VirtualHosts>
		    <VirtualHost>
            <Applications>
                <Application>
                    <Publishers>
                        <HLS>
                            <SegmentDuration>5</SegmentDuration>
                            <SegmentCount>3</SegmentCount>
                            <CrossDomain>
                                <Url>*<Url>
                            </CrossDomain>
                        </HLS>
                        <DASH>
                            <SegmentDuration>5</SegmentDuration>
                            <SegmentCount>3</SegmentCount>
                            <CrossDomain>
                                <Url>*<Url>
                            </CrossDomain>
                        </DASH>
                        <LLDASH>
                            <SegmentDuration>5</SegmentDuration>
                            <CrossDomain>
                                <Url>*<Url>
                            </CrossDomain>
                        </LLDASH>
                    </Publishers>
                </Application>
            </Applications>
        </VirtualHost>
    </VirtualHosts>
</Server>
```

<table>
  <thead>
    <tr>
      <th style="text-align:left">Element</th>
      <th style="text-align:left">Decsription</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Bind</td>
      <td style="text-align:left">Set the HTTP port to provide HLS and DASH. LLDASH and DASH are provided
        on the same port, and DASH and HLS can be set to the same port.</td>
    </tr>
    <tr>
      <td style="text-align:left">SegmentDuration</td>
      <td style="text-align:left">Set the length of the segment in seconds. The shorter the <code>&lt;SegmentDuration&gt;</code>,
        the lower latency can be streamed, but it is less stable during streaming.
        Therefore, we are recommended to set it to 3 to 5 seconds.</td>
    </tr>
    <tr>
      <td style="text-align:left">SegmentCount</td>
      <td style="text-align:left">
        <p>Set the number of segments to be exposed to <code>*.mpd</code>. The smaller
          the number of <code>&lt;SegmentCount&gt;</code>, the lower latency can be
          streamed, but it is less reliable during streaming. Therefore, it is recommended
          to set this value to 3.</p>
        <p>It does not need to set SegmentCount for LLDASH because LLDASH only has
          one segment on OvenMediaEngine.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">CrossDomain</td>
      <td style="text-align:left">Control the domain in which the player works through <code>&lt;CorssDomain&gt;</code>.
        For more information, please refer to the <a href="hls-mpeg-dash.md#crossdomain">CrossDomain</a> section.</td>
    </tr>
  </tbody>
</table>

## CrossDomain

Most browsers and players prohibit accessing other domain resources in the currently running domain. You can control this situation through Cross-Origin Resource Sharing \(CORS\) or Cross-Domain \(CrossDomain\). You can set CORS and Cross-Domain as `<CrossDomain>` element.

{% code title="Server.xml" %}
```markup
<CrossDomain>
    <Url>*</Url>
    <Url>*.airensoft.com</Url>
    <Url>http://*.ovenplayer.com</Url>
    <Url>https://demo.ovenplayer.com</Url>
</CrossDomain>
```
{% endcode %}

You can set it using the `<Url>` element as shown above, and you can use the following values:

| Url Value | Decsription |
| :--- | :--- |
| \* | Allows requests from all Domains |
| domain | Allows both HTTP and HTTPS requests from the specified Domain |
| http://domain | Allows HTTP requests from the specified Domain |
| https://domain | Allows HTTPS requests from the specified Domain |

## Streaming

LLDASH, DASH and HLS Streaming are ready when a live source is inputted and a stream is created. Viewers can stream using OvenPlayer or other players.

Also, you need to set H.264 and AAC in the Transcoding profile because MPEG-DASH and HLS use these codecs.

```markup
<Encodes>
   <Encode>
      <Name>HD_H264_AAC</Name>
      <Audio>
         <Codec>aac</Codec>
         <Bitrate>128000</Bitrate>
         <Samplerate>48000</Samplerate>
         <Channel>2</Channel>
      </Audio>
      <Video>
         <Codec>h264</Codec>
         <Width>1280</Width>
         <Height>720</Height>
         <Bitrate>2000000</Bitrate>
         <Framerate>30.0</Framerate>
      </Video>
   </Encode>
   <Encode>
      <Name>BYPASS</Name>
      <Audio>
         <Bypass>true</Bypass>
      </Audio>
      <Video>
         <Bypass>true</Bypass>
      </Video>
   </Encode>
</Encodes>
<Streams>
   <Stream>
      <Name>${OriginStreamName}_o</Name>
      <Profiles>
         <Profile>HD_H264_AAC</Profile>
      </Profiles>
   </Stream>
   <Stream>
      <Name>${OriginStreamName}_bypass</Name>
      <Profiles>
         <Profile>BYPASS</Profile>
      </Profiles>
   </Stream>
</Streams>
```

When you create a stream as shown above, you can play LLDASH, DASH and HLS through OvenPlayer with the following URL:

| Protocol | URL format |
| :--- | :--- |
| LLDASH | `http://<Server IP>[:<DASH Port]/<Application Name>/<Stream Name>/manifest_ll.mpd` |
| Secure LLDASH | `https://<Domain>[:<DASH Port]/<Application Name>/<Stream Name>/manifest_ll.mpd` |
| DASH | `http://<Server IP>[:<DASH Port]/<Application Name>/<Stream Name>/manifest.mpd` |
| Secure DASH | `https://<Domain>[:<DASH Port]/<Application Name>/<Stream Name>/manifest.mpd` |
| HLS | `http://<Server IP>[:<HLS Port]/<Application Name>/<Stream Name>/playlist.m3u8` |
| Secure HLS | `https://<Domain>[:<HLS Port]/<Application Name>/<Stream Name>/playlist.m3u8` |

If you use the default configuration, you can start streaming with the following URL:

* `https://[OvenMediaEngine IP]:80/app/stream_o/playlist.m3u8`
* `http://[OvenMediaEngine IP]:80/app/stream_o/playlist.m3u8`
* `https://[OvenMediaEngine IP]:80/app/stream_o/manifest.mpd`
* `http://[OvenMediaEngine IP]:80/app/stream_o/manifest.mpd`
* `https://[OvenMediaEngine IP]:80/app/stream_o/manifest_ll.mpd`
* `http://[OvenMediaEngine IP]:80/app/stream_o/manifest_ll.mpd`

We have prepared a test player that you can quickly see if OvenMediaEngine is working. Please refer to the [Test Player](test-player.md) for more information.
