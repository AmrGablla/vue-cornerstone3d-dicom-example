# Simple Vue 3 App with Cornerstone3D DICOM P10 Example

This repository contains a simple Vue 3 application that utilizes Cornerstone3D to display DICOM P10 files from the local file system. It wraps the Cornerstone3D library with Vue components to provide an interactive and user-friendly interface for viewing medical images.

## Prerequisites

Before getting started, ensure you have the following tools and technologies installed:

- Node.js (>= 14.x)
- Vue CLI (Installation: `npm install -g @vue/cli`)

## Getting Started

1. Clone this repository:

   ```bash
   git clone https://github.com/AmrGablla/vue-cornerstone3d-dicom-example.git
   cd vue-cornerstone3d-dicom-example
   ```

2. Install dependencies:

   ```bash
   npm install
   ```

3. Run the development server:

   ```bash
   npm run serve
   ```

4. Open your browser and navigate to `http://localhost:8080` to see the app in action.

## Usage

1. Launch the app, and you will be presented with a simple interface for uploading DICOM P10 files.

2. Click the "Choose File" button to select a DICOM P10 file from your local file system.

3. Once a file is selected, the app will display the medical image using Cornerstone3D's viewer component.

## Cornerstone3D and DICOM P10

Cornerstone3D is a library that integrates with Cornerstone, a popular JavaScript library for medical image display. It adds support for rendering medical images in 3D using WebGL. DICOM P10 is a format commonly used for storing medical images and associated metadata.

## License

This project is licensed under the [MIT License](LICENSE).

## Acknowledgments

- This app was created with the help of the Vue CLI.
- Cornerstone3D and Cornerstone libraries are used to handle the rendering of medical images.
- DICOM P10 files are widely used in the medical imaging field for storing and exchanging medical images.

Feel free to customize this README to better suit your project's specific details and requirements.
