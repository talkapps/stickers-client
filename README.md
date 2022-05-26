# Stickers Client

## Rationale

Communicating with Signal's API is non-trivial because it uses encrypted
[protocol buffers](https://en.wikipedia.org/wiki/Protocol_Buffers). Responses
must be parsed accordingly and their contents decrypted using some rather
esoteric cryptography. This package aims to make using the stickers API as
straightforward as possible by providing a small set of functions that work in
both the browser and Node.

## Install

```
npm i @signalstickers/stickers-client
```

## Use

It is assumed that users of this package have a general understanding of how
stickers work in Signal. For more information on that topic, see
[this article](https://support.signal.org/hc/en-us/articles/360031836512-Stickers).

This package can be used in both browser and Node environments. If you plan to
use it in the browser, make sure you are using a bundler that supports the
[`browser` `package.json` field](https://github.com/defunctzombie/package-browser-field-spec).

Because sticker packs are immutable, responses from Signal can be safely cached
indefinitely. As such, this package implements a basic in-memory cache. This
reduces the amount of superfluous networks requests made.

This package has the following named exports:

#### `getStickerPackManifest(id: string, key: string): Promise<StickerPackManifest>`

Provided a sticker pack ID and its key, returns a promise that resolves with the
sticker pack's decrypted manifest.

<a href="#top"><img src="https://user-images.githubusercontent.com/441546/72722991-8988bf00-3b34-11ea-8fff-b9b1dfaa0a53.png"></a>

#### `getStickerInPack(id: string, key: string, stickerId: number, encoding? = 'raw' | 'base64): Promise<Uint8Array | string>`

Provided a sticker pack ID, its key, and a sticker ID, returns a promise that
resolves with the raw WebP image data for the indicated sticker.

An optional `encoding` parameter may be provided to indicate the desired return
type. The default value of `raw` will return raw WebP data as a `Uint8` Array.
This is useful if further processing of the image data is necessary.
Alternatively, if `base64` is provided, a data-URI `string` will be returned
instead. This string can be used directly as the `src` attribute in an `<img>`
tag, for example.

<a href="#top"><img src="https://user-images.githubusercontent.com/441546/72722991-8988bf00-3b34-11ea-8fff-b9b1dfaa0a53.png"></a>

#### `getEmojiForSticker(id: string, key: string, stickerId: number): Promise<string>`

Provided a sticker pack ID, its key, and sticker ID, returns the emoji
associated with the sticker.

<a href="#top"><img src="https://user-images.githubusercontent.com/441546/72722991-8988bf00-3b34-11ea-8fff-b9b1dfaa0a53.png"></a>

**Example**

In this example, we'll create a simple React component that will render a single
sticker. It will accept a sticker pack's ID and key as well as the ID of the
sticker to render as props. We will use a one-time effect when the component
mounts to fetch the image.

Note that this package caches responses from Signal, so we don't need to add any
additional logic to store the result of `getStickerInPack` outside the
component's state. If it is ever dismounted and remounted in the future with the
same props, no additional network requests will be made.

```tsx
import React, {useEffect, useState} from 'react';
import {getStickerInPack, getEmojiForSticker} from '@signalstickers/stickers-client';

export interface Props {
  packId: string;
  packKey: string;
  stickerId: number;
}

const StickerComponent: React.FunctionComponent<Props> = ({packId, packKey, stickerId}) => {
  const [stickerData, setStickerData] = useState();
  const [stickerEmoji, setStickerEmoji] = useState();

  useEffect(() => {
    getStickerInPack(packId, packKey, stickerId, 'base64').then(setStickerData);
    getEmojiForSticker(packId, packKey, stickerId).then(setStickerEmoji);
  }, []);

  if (!stickerData || !stickerEmoji) {
    return null;
  }

  return (
    <img src={stickerData} alt={stickerEmoji} />
  );
};

export default StickerComponent;
```
