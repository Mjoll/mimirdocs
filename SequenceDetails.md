# Retrieve detailed information about Premiere sequence exported via Mimir
Details about a sequence i.e. clips used, timing information etc. are stored inside a JSON file. This file can be retrieved using the `get item` API endpoint. The steps to do this are as follows:

1. Specify the property 'exportedSequenceDetailsUrl' in the query parameter `fields` of the 'get item' endpoint. This property is not returned by default, only when it's asked for.

```
curl 'https://mimir.mjoll.no/api/v1/items/<item-id>?fields=exportedSequenceDetailsUrl' \
-H 'JWT_TOKEN' \

--compressed
```

2. If the item is an exported sequence, there will be a property `exportedSequenceDetailsUrl` in the response body. The value of the property is a signed URL to a JSON file.

3. Doing a `GET` on that URL will return a JSON object containing information about all the media tracks in the sequence and corresponding clips in each track.

### Details about the different properties in the JSON object:

```
{
    "projectType": "prproj",
    "tracks": [
        {
            "id": 1, // Number assigned to this track by Premiere. Example: 1, 2
            "name": "Video 1", // Name of this track
            "mediaType": "video", // Media type of this track. Can be 'video' or 'audio'
           
            // A list of clips in this track
            "clips": [{
                "mimirItemId": "<item-id>", // ID of the Mimir item

                // Starting and end time of the track item in the sequence (in milliseconds)
                "start": 0,
                "end": 17000,

                "duration": 17000, // Total duration of the tack (end - start)
               
                // Premiere's interpretation of the start and end times of the source media (in milliseocnds).
                // Relevant if this is a subclip of another file.
                "inPointPremiere": 0,
                "outPointPremiere": 17000,

                // Mimir's interpretation of the start and end times of the source media (in milliseconds).
                // This can be different from Premiere's interpretation if the clip is sped up or reversed.
                // These values are not effected by those factors.
                "inPoint": 0,
                "outPoint": 17000,

                "speed": 1, // Relevant if clip was sped up. Default is 1.
                "isSpeedReversed": false, // Whether the clip is supposed to play backwards
                "mediaPath": "/Users/username/media-folder/file.mp4", // Local path of the source media when exported
                "key": "sequence-key" // A key assigned by Mimir for bookkeeping
            }],
            "trackLabel": "V1" // Synthesized from track type and order. Example: 'V1', 'A2' 
        }
    ],

    // Sequence start and end point in milliseconds. These values will be positive only if mark in and out points were selected
    // in the sequence timeline when exporting.
    "sequenceInPointInMs": -400000000,
    "sequenceOutPointInMs": -400000000
}
```

