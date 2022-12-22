# QR Code Styling
[![Version](https://img.shields.io/npm/v/qr-code-styling.svg)](https://www.npmjs.org/package/qr-code-styling)

JavaScript library for generating QR codes with a logo and styling.

Try it here https://qr-code-styling.com

# Library was modified to be used inside Web Worker
For more info message to spacehaz@gmail


## Create helper to load image
This code should be used outside of worker. It is needed to generate an image with Image class and define actual size of logo in the center of future QR-code

```tsx
// load-image.tsx
type TQRImageOptions = {
  hideBackgroundDots?: boolean;
  imageSize?: number;
  crossOrigin?: string;
  margin?: number;
}

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
You need to get an image bitmap for logo. It can be used inside virtual canvas, so you can pass it to QRCodeStyling class as parameter. Also you need to generate logo once and get width and height of that logo. That data can also be passed to QRCodeStyling class as parameter

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

  // here should be the call of createQR function or any method of your worker
  const myQRBlob = await createQR(
    qrImageOptions,
    logoImageLoaded.width,
    logoImageLoaded.height,
    img
  )
}

```

## Create worker
Use provided example to generate QR-code with image bitmap and sizes

```tsx

const createQR = await (
  qrImageOptions: TQRImageOptions,
  logoImageWidth: number,
  logoImageHeight: number,
  img: ImageBitmap,
) => {
  const qrCode = new QRCodeStyling({
    data: `https://linkdrop.io`,
    width, // width of QR-code
    height, // height of QR-code
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
    logoImageWidth, // width of logo image
    logoImageHeight // height of logo image
  })

  const blob = await qrCode.getRawData('png')
  
  return blob
}


```