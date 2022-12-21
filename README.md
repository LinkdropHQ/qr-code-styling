# QR Code Styling
[![Version](https://img.shields.io/npm/v/qr-code-styling.svg)](https://www.npmjs.org/package/qr-code-styling)

JavaScript library for generating QR codes with a logo and styling.

Try it here https://qr-code-styling.com

# Library was modified to be used inside Web Worker
For more info message to spacehaz@gmail


## Create helper to load image on browser side
This code should be outside of worker

```tsx
// load-image.tsx
import { TQRImageOptions } from 'types'

type TLoadImage = (
  imageOptions: TQRImageOptions,
  imageSrc: string
) => Promise<HTMLImageElement>

const loadImage: TLoadImage = (
  imageOptions,
  imageSrc
) => {
  return new Promise((resolve, reject) => {
    const image = new Image();

    if (!image) {
      return reject("Image is not defined");
    }

    if (typeof imageOptions.crossOrigin === "string") {
      image.crossOrigin = imageOptions.crossOrigin;
    }

    image.onload = (): void => {
      resolve(image);
    };

    image.src = imageSrc;
  });
}

export default loadImage

```

## Create QR code with previously rendered logo using helper above

```tsx
import LedgerIcon from 'images/sample-logo.png' // logo for QR
import loadImage from './load-image.tsx'
import QRCodeStyling from 'qr-code-styling-bigmac'

const initialize = await () => {

  const resp = await fetch(LedgerIcon)
  const blob = await resp.blob()
  const img = await createImageBitmap(blob as ImageBitmapSource) // create image bitmap for logo

  const qrImageOptions = {
    margin: 1,
    imageSize: 0.5,
    crossOrigin: 'anonymous',
  }

  const logoImageLoaded = await loadImage(
    qrImageOptions,
    LedgerIcon
  ) // generate image outside of worker to define image actual size

  const qrCode = new QRCodeStyling({
    data: `https://linkdrop.io`,
    width, // width of qr
    height, // height of qr
    margin: width / 60,
    type: 'canvas',
    cornersSquareOptions: {
      color: "#FFF",
      type: 'square'
    },
    cornersDotOptions: {
      color: "#FFF",
      type: 'square'
    },
    dotsOptions: {
      color: "#FFF",
      type: "dots"
    },
    backgroundOptions: {
      color: "#000",
    },
    image: img, // image of logo in center itself
    imageOptions: qrImageOptions,
    logoImageWidth: logoImageLoaded.width, // width of logo image
    logoImageHeight: logoImageLoaded.height // height of logo image
  })

}



```