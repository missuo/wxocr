# WeChat OCR API Docker

A Dockerized REST API service for text recognition using WeChat's OCR engine.

## Overview

This project wraps the WeChat OCR functionality from the excellent [wechat-ocr](https://github.com/swigger/wechat-ocr) project into a simple REST API service that can be easily deployed using Docker. It allows you to perform optical character recognition on images by leveraging WeChat's powerful OCR capabilities.

## ⚠️ Warning

**This is an open-source project intended for learning and communication purposes only. Please do not use it for commercial activities. Users are solely responsible for any consequences resulting from improper use of this project.**

## Acknowledgements

This project would not be possible without the work of [swigger](https://github.com/swigger) and their [wechat-ocr](https://github.com/swigger/wechat-ocr) project. Their efforts in reverse-engineering and creating a usable interface for WeChat's OCR functionality form the foundation of this service.

## Quick Start

### Using Docker

```bash
# Git clone the repository
git clone https://github.com/missuo/wxocr.git

# Enter the repository
cd wxocr

# Run the container
docker compose up -d --build
```

### API Usage

Send a POST request to `/ocr` with a JSON payload containing your base64-encoded image:

```bash
curl -X POST http://localhost:5000/ocr \
  -H "Content-Type: application/json" \
  -d '{"image": "BASE64_ENCODED_IMAGE_DATA"}'
```

#### Example Response

```json
{
  "errcode": 0,
  "height": 72,
  "width": 410,
  "imgpath": "temp/5726fe7b-25d6-43a6-a50d-35b5f668fbb6.png",
  "ocr_response": [
    {
      "text": "aacss",
      "left": 80.63632202148438,
      "top": 29.634929656982422,
      "right": 236.47093200683594,
      "bottom": 55.28932189941406,
      "rate": 0.9997046589851379
    },
    {
      "text": "xxzsa",
      "left": 312.625,
      "top": 30.75,
      "right": 395.265625,
      "bottom": 55.09375,
      "rate": 0.997739315032959
    }
  ]
}
```

### Python Client Example

Here's a simple Python client to use the OCR API:

```python
import requests
import base64
import os

def ocr_recognize(image_path=None, image_url=None, api_url="http://localhost:5000/ocr"):
    """
    Send an image to the OCR API service and get the recognition results.
    Use either image_path or image_url (one is required).
    """
    # Get image data
    if image_path:
        if not os.path.exists(image_path):
            print(f"Error: Local image not found: {image_path}")
            return
        with open(image_path, "rb") as image_file:
            img_data = image_file.read()
    elif image_url:
        try:
            response = requests.get(image_url)
            response.raise_for_status()
            img_data = response.content
        except Exception as e:
            print(f"Failed to download image: {str(e)}")
            return
    else:
        print("Please provide either image_path or image_url")
        return

    # Convert image to base64
    base64_image = base64.b64encode(img_data).decode('utf-8')
    
    # Send request to API
    try:
        response = requests.post(api_url, json={"image": base64_image})
        response.raise_for_status()
        return response.json()
    except Exception as e:
        print(f"API request failed: {str(e)}")
        return None

# Example usage
if __name__ == "__main__":
    # Local image example
    result = ocr_recognize(image_path="ocrtest.png")
    if result:
        print(result)
    
    # URL image example (uncomment to use)
    # result = ocr_recognize(image_url="https://example.com/image.png")
```

## Project Structure

- `main.py`: The Flask API service that handles OCR requests
- `opt/wechat/wxocr`: WeChat OCR binary
- `opt/wechat/`: WeChat runtime dependencies

## Technical Details

This service uses a Flask application to provide a REST API interface to the WeChat OCR functionality. When an image is submitted:

1. The base64-encoded image is decoded
2. A temporary file is created
3. The image is processed by the WeChat OCR engine via the wcocr Python binding
4. Results are returned in JSON format
5. Temporary files are cleaned up

## Limitations

- Currently only supports PNG images (can be extended if needed)
- Depends on WeChat's OCR binaries which may be updated by WeChat

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
