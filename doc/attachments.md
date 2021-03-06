# What does the Data URI format using Base64 string look like?

## On iOS devices

Your full base64 string must look like the following:

```
base64:filename.ext//yourbase64datahere
```

## Why exactly?

Look at the following code below for file `APPEmailComposerImpl.m`:

```ObjectiveC
/**
 * Returns the data for a given (relative) attachment path.
 *
 * @param path An absolute/relative path or the base64 data
 *
 * @return The data for the attachment.
 */
- (NSData*) getDataForAttachmentPath:(NSString*)path
{
    if ([path hasPrefix:@"file:///"])
    {
        return [self dataForAbsolutePath:path];
    }
    else if ([path hasPrefix:@"res:"])
    {
        return [self dataForResource:path];
    }
    else if ([path hasPrefix:@"file://"])
    {
        return [self dataForAsset:path];
    }
    else if ([path hasPrefix:@"app://"])
    {
        return [self dataForAppInternalPath:path];
    }
    else if ([path hasPrefix:@"base64:"])
    {
        return [self dataFromBase64:path];
    }

    NSFileManager* fm = [NSFileManager defaultManager];

    if (![fm fileExistsAtPath:path]){
        NSLog(@"File not found: %@", path);
    }

    return [fm contentsAtPath:path];
}
```

The following code points out that for your string to be recognized as a base 64 data representation of the file to be attached, it must begin with the prefix `base64:`.

## Filename

After the base64 prefix, you must specify the file name. 

```
- (NSString*) getBasenameFromAttachmentPath:(NSString*)path
{
    if ([path hasPrefix:@"base64:"])
    {
        NSString* pathWithoutPrefix;

        pathWithoutPrefix = [path stringByReplacingOccurrencesOfString:@"base64:"
                                                            withString:@""];

        return [pathWithoutPrefix substringToIndex:
                [pathWithoutPrefix rangeOfString:@"//"].location];
    }

    return path;
}
```

From reading the above code, we recognize that it attempts to find the filename between the `base64:` part and the `//` part. This means that the filename must follow right after the base64 prefix, and it must be delimited using `//`.
