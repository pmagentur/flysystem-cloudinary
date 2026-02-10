# Flysystem Adapter for Cloudinary

[![Build Status](https://img.shields.io/travis/enl/flysystem-cloudinary/master.svg?style=flat-square)](https://travis-ci.org/enl/flysystem-cloudinary)
[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE)
[![Coverage Status](https://coveralls.io/repos/enl/flysystem-cloudinary/badge.svg?branch=master&service=github&style=flat-square)](https://coveralls.io/github/enl/flysystem-cloudinary?branch=master)

A powerful [Flysystem adapter](https://github.com/thephpleague/flysystem) for the [Cloudinary API](http://cloudinary.com/documentation/php_integration), providing seamless integration between Flysystem's filesystem abstraction and Cloudinary's cloud-based image and video management service.

## Table of Contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Usage Examples](#usage-examples)
- [Advanced Features](#advanced-features)
  - [Image Transformations](#image-transformations)
  - [Path Converters](#path-converters)
  - [Available Plugins](#available-plugins)
- [Cloudinary Specific Behaviors](#cloudinary-specific-behaviors)
- [API Reference](#api-reference)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## Features

- ✅ Full Flysystem API support for Cloudinary
- ✅ Upload images and videos to Cloudinary
- ✅ Read, update, and delete files
- ✅ List directory contents
- ✅ Get file metadata (size, mimetype, timestamp)
- ✅ Support for Cloudinary transformations
- ✅ Customizable path converters
- ✅ Video file support
- ✅ Overwrite protection
- ✅ Automatic folder creation

## Requirements

- PHP >= 5.4.0
- ext-fileinfo
- league/flysystem ^1.0.26
- cloudinary/cloudinary_php ^1.8.0

## Installation

Install the package via Composer:

```bash
composer require pmagentur/flysystem-cloudinary
```

Or add it to your `composer.json`:

```json
{
    "require": {
        "pmagentur/flysystem-cloudinary": "^1.0"
    }
}
```

Then run:

```bash
composer update
```

## Quick Start

Here's a minimal example to get you started:

```php
<?php
use Enl\Flysystem\Cloudinary\ApiFacade as CloudinaryClient;
use Enl\Flysystem\Cloudinary\CloudinaryAdapter;
use League\Flysystem\Filesystem;

require __DIR__ . '/vendor/autoload.php';

// Create Cloudinary client with your credentials
$client = new CloudinaryClient([
    'cloud_name' => 'your-cloud-name',
    'api_key' => 'your-api-key',
    'api_secret' => 'your-api-secret',
    'overwrite' => false,
]);

// Create the adapter and filesystem
$adapter = new CloudinaryAdapter($client);
$filesystem = new Filesystem($adapter);

// Upload a file
$filesystem->write('path/to/image.jpg', file_get_contents('local/image.jpg'));

// Read a file
$contents = $filesystem->read('path/to/image.jpg');

// Check if file exists
if ($filesystem->has('path/to/image.jpg')) {
    echo "File exists!";
}

// Delete a file
$filesystem->delete('path/to/image.jpg');
```

## Configuration

### Basic Configuration

The `ApiFacade` constructor accepts an array of configuration options:

```php
$client = new CloudinaryClient([
    'cloud_name' => 'your-cloud-name',  // Required: Your Cloudinary cloud name
    'api_key' => 'your-api-key',        // Required: Your API key
    'api_secret' => 'your-api-secret',  // Required: Your API secret
    'overwrite' => true,                // Optional: Enable file overwriting (default: false)
]);
```

### Configuration for File Overwriting

If you want to overwrite existing files using `$filesystem->write()`, you need to:

1. Set `overwrite` to `true` in the Cloudinary client configuration
2. Set `disable_asserts` to `true` in the Filesystem configuration

```php
$client = new CloudinaryClient([
    'cloud_name' => 'your-cloud-name',
    'api_key' => 'your-api-key',
    'api_secret' => 'your-api-secret',
    'overwrite' => true,
]);

$adapter = new CloudinaryAdapter($client);
$filesystem = new Filesystem($adapter, ['disable_asserts' => true]);
```

### Delete Options

You can configure options for delete operations:

```php
$client = new CloudinaryClient([
    'cloud_name' => 'your-cloud-name',
    'api_key' => 'your-api-key',
    'api_secret' => 'your-api-secret',
]);

// Set delete options (e.g., invalidate CDN cache on delete)
$client->setDeleteOptions(['invalidate' => true]);
```

## Usage Examples

### Uploading Files

```php
// Upload from local file
$filesystem->write('images/photo.jpg', file_get_contents('/path/to/local/photo.jpg'));

// Upload with metadata
$filesystem->writeStream('videos/movie.mp4', fopen('/path/to/video.mp4', 'r'));
```

### Reading Files

```php
// Read file contents
$contents = $filesystem->read('images/photo.jpg');

// Read as stream
$stream = $filesystem->readStream('images/photo.jpg');
```

### File Operations

```php
// Check if file exists
$exists = $filesystem->has('images/photo.jpg');

// Get file metadata
$metadata = $filesystem->getMetadata('images/photo.jpg');
$size = $filesystem->getSize('images/photo.jpg');
$mimetype = $filesystem->getMimetype('images/photo.jpg');
$timestamp = $filesystem->getTimestamp('images/photo.jpg');

// Delete file
$filesystem->delete('images/photo.jpg');

// Rename/Move file
$filesystem->rename('images/old-name.jpg', 'images/new-name.jpg');

// Copy file
$filesystem->copy('images/source.jpg', 'images/destination.jpg');
```

### Working with Directories

```php
// List directory contents
$contents = $filesystem->listContents('images');

// List recursively
$contents = $filesystem->listContents('images', true);

// Delete directory (note: Cloudinary doesn't support empty directory deletion)
$filesystem->deleteDir('images/old-folder');
```

## Advanced Features

### Image Transformations

Cloudinary's powerful image transformation features are available through Flysystem plugins. See the [transformations documentation](doc/transformations.md) for full details.

#### Setting Up Transformation Plugins

```php
use Enl\Flysystem\Cloudinary\Plugin\GetUrl;
use Enl\Flysystem\Cloudinary\Plugin\ReadTransformation;

$filesystem->addPlugin(new GetUrl($client));
$filesystem->addPlugin(new ReadTransformation($client));
```

#### Getting Transformed Image URLs

```php
// Get URL with transformations
$url = $filesystem->getUrl('image.jpg', [
    'width' => 600,
    'height' => 400,
    'crop' => 'fill',
    'gravity' => 'face',
    'format' => 'png'
]);

// More transformation examples
$thumbnailUrl = $filesystem->getUrl('image.jpg', [
    'width' => 150,
    'height' => 150,
    'crop' => 'thumb',
    'gravity' => 'face'
]);

$responsiveUrl = $filesystem->getUrl('image.jpg', [
    'width' => 'auto',
    'dpr' => 'auto',
    'responsive' => true
]);
```

#### Reading Transformed Images

```php
// Get transformed image content
$transformedContent = $filesystem->readTransformation('image.jpg', [
    'width' => 600,
    'height' => 400,
    'effect' => 'grayscale'
]);

// Save transformed image locally
file_put_contents('local-transformed.jpg', $transformedContent);
```

### Path Converters

Path converters control how file paths are mapped to Cloudinary public IDs. See the [path converter documentation](doc/path_converter.md) for full details.

#### Using the TruncateExtensionConverter

By default, the adapter uses `AsIsPathConverter`, which keeps paths as-is. To automatically handle file extensions:

```php
use Enl\Flysystem\Cloudinary\ApiFacade;
use Enl\Flysystem\Cloudinary\Converter\TruncateExtensionConverter;

$client = new ApiFacade([
    'cloud_name' => 'your-cloud-name',
    'api_key' => 'your-api-key',
    'api_secret' => 'your-api-secret',
], new TruncateExtensionConverter());
```

The `TruncateExtensionConverter` removes file extensions from paths when creating public IDs and restores them when retrieving files.

#### Creating Custom Path Converters

```php
use Enl\Flysystem\Cloudinary\Converter\PathConverterInterface;

class CustomPathConverter implements PathConverterInterface
{
    public function pathToId($path)
    {
        // Convert path to Cloudinary public_id
        return str_replace('/', '-', $path);
    }

    public function idToPath($resource)
    {
        // Convert Cloudinary resource back to path
        $publicId = $resource['public_id'];
        return str_replace('-', '/', $publicId);
    }
}

$client = new ApiFacade($options, new CustomPathConverter());
```

### Available Plugins

#### GetUrl Plugin

Get the public URL of a file with optional transformations:

```php
use Enl\Flysystem\Cloudinary\Plugin\GetUrl;

$filesystem->addPlugin(new GetUrl($client));
$url = $filesystem->getUrl('image.jpg', ['width' => 300, 'height' => 300]);
```

#### ReadTransformation Plugin

Read the content of a transformed image:

```php
use Enl\Flysystem\Cloudinary\Plugin\ReadTransformation;

$filesystem->addPlugin(new ReadTransformation($client));
$content = $filesystem->readTransformation('image.jpg', ['effect' => 'sepia']);
```

#### GetVersionedUrl Plugin

Get the URL of a specific version of a file:

```php
use Enl\Flysystem\Cloudinary\Plugin\GetVersionedUrl;

$filesystem->addPlugin(new GetVersionedUrl($client));
$url = $filesystem->getVersionedUrl('image.jpg', '1234567890');
```

## Cloudinary Specific Behaviors

When working with Cloudinary through this adapter, keep in mind these platform-specific behaviors:

### File Extension Handling

**Important:** Cloudinary automatically appends file extensions to public IDs. This can cause unexpected behavior:

```php
// If you upload with public_id 'test.jpg'
$filesystem->write('test.jpg', $content);

// Cloudinary will store it as 'test.jpg.jpg'
```

**Solution:** Use a [PathConverter](#path-converters) to handle this automatically:

```php
use Enl\Flysystem\Cloudinary\Converter\TruncateExtensionConverter;

$client = new ApiFacade($options, new TruncateExtensionConverter());
```

### Folder Creation

Cloudinary **does not support** creating empty folders through the API. Folders are created automatically when you upload files with paths:

```php
// This creates the 'images/products' folder structure automatically
$filesystem->write('images/products/item.jpg', $content);
```

**Note:** To enable automatic folder creation, ensure this setting is enabled in your Cloudinary dashboard:
- Go to Settings → Upload → Auto-create folders

### Folder Organization

To organize files in folders, include the folder path in the file name:

```php
// Good - creates folder structure
$filesystem->write('images/2024/january/photo.jpg', $content);
$filesystem->write('documents/reports/annual-report.pdf', $content);

// These files will be organized in the Cloudinary media library
```

### Video File Support

The adapter automatically detects and handles video files based on their extension. Supported video formats include:

- 3gp, 3g2
- avi
- flv
- mov, mkv
- mp4, mpeg
- webm, wmv
- And more...

```php
// Upload video file
$filesystem->write('videos/presentation.mp4', file_get_contents('local-video.mp4'));

// Video files are stored with resource_type 'video' in Cloudinary
```

### Public URLs

All files uploaded to Cloudinary are **publicly accessible** by default. This adapter does not support visibility settings (private/public) as they are not part of Cloudinary's standard API.

### File Overwrites

By default, Cloudinary **prevents** overwriting existing files. To enable overwrites:

```php
$client = new CloudinaryClient([
    'cloud_name' => 'your-cloud-name',
    'api_key' => 'your-api-key',
    'api_secret' => 'your-api-secret',
    'overwrite' => true,  // Enable overwriting
]);

$filesystem = new Filesystem($adapter, ['disable_asserts' => true]);
```

## API Reference

### CloudinaryAdapter Methods

The adapter implements the standard Flysystem `AdapterInterface`:

```php
// Write operations
write($path, $contents, Config $config)
writeStream($path, $resource, Config $config)
update($path, $contents, Config $config)
updateStream($path, $resource, Config $config)

// Read operations
read($path)
readStream($path)
has($path)

// Metadata operations
getMetadata($path)
getSize($path)
getMimetype($path)
getTimestamp($path)

// File operations
rename($path, $newpath)
copy($path, $newpath)
delete($path)

// Directory operations
listContents($directory = '', $recursive = false)
createDir($dirname, Config $config)  // Note: No-op in Cloudinary
deleteDir($dirname)
```

### ApiFacade Methods

Additional methods provided by the `ApiFacade` class:

```php
// Configure Cloudinary
configure(array $options)

// Set delete options
setDeleteOptions(array $options)

// Upload file
uploadFile($path, $contents, array $options = [])

// Delete file
deleteFile($path, array $options = [])

// Get resource information
resource($path, array $options = [])

// List resources
resources(array $options = [])
```

## Troubleshooting

### Common Issues

#### "File already exists" error when uploading

**Problem:** You get an error when trying to upload a file that already exists.

**Solution:** Enable the `overwrite` option and `disable_asserts`:

```php
$client = new CloudinaryClient([
    'cloud_name' => 'your-cloud-name',
    'api_key' => 'your-api-key',
    'api_secret' => 'your-api-secret',
    'overwrite' => true,
]);

$filesystem = new Filesystem($adapter, ['disable_asserts' => true]);
```

#### Files have double extensions (e.g., image.jpg.jpg)

**Problem:** Uploaded files have their extension duplicated.

**Solution:** Use the `TruncateExtensionConverter`:

```php
use Enl\Flysystem\Cloudinary\Converter\TruncateExtensionConverter;

$client = new ApiFacade($options, new TruncateExtensionConverter());
```

#### Cannot create folders

**Problem:** Trying to create empty folders fails.

**Solution:** Cloudinary doesn't support empty folders. Upload files with folder paths instead:

```php
// Don't do this
$filesystem->createDir('images/new-folder');  // This won't work

// Do this instead
$filesystem->write('images/new-folder/placeholder.jpg', $content);
```

#### Authentication errors

**Problem:** "401 Unauthorized" or authentication errors.

**Solution:** Verify your credentials are correct:
- Check your Cloudinary dashboard for correct `cloud_name`, `api_key`, and `api_secret`
- Ensure there are no extra spaces in your credentials
- Verify your Cloudinary account is active

#### Slow upload performance

**Problem:** Uploads are taking too long.

**Solution:** 
- Use `writeStream()` for large files instead of `write()`
- Consider uploading to Cloudinary directly from URLs when possible
- Check your network connection and Cloudinary service status

### Debugging

Enable Cloudinary logging to debug issues:

```php
// Enable Cloudinary debug mode
\Cloudinary::config([
    'cloud_name' => 'your-cloud-name',
    'api_key' => 'your-api-key',
    'api_secret' => 'your-api-secret',
]);

// Log all Cloudinary API calls
error_log(print_r(\Cloudinary::config_get(), true));
```

### Getting Help

If you encounter issues:

1. Check the [Cloudinary documentation](https://cloudinary.com/documentation)
2. Review [Flysystem documentation](https://flysystem.thephpleague.com/)
3. Open an issue on [GitHub](https://github.com/pmagentur/flysystem-cloudinary/issues)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

### Development Setup

```bash
# Clone the repository
git clone https://github.com/pmagentur/flysystem-cloudinary.git
cd flysystem-cloudinary

# Install dependencies
composer install

# Run tests
vendor/bin/phpunit

# Run code style checks
vendor/bin/phpcs
```

### Running Tests

```bash
# Run all tests
composer test

# Run specific test
vendor/bin/phpunit tests/YourTest.php
```

## License

This package is open-sourced software licensed under the [MIT license](LICENSE).

## Credits

- Original author: Alex Panshin
- Current maintainer: pmagentur
- Built on top of [Flysystem](https://github.com/thephpleague/flysystem) by [The PHP League](https://thephpleague.com/)
- Uses [Cloudinary PHP SDK](https://github.com/cloudinary/cloudinary_php)

## Related Links

- [Flysystem Documentation](https://flysystem.thephpleague.com/)
- [Cloudinary Documentation](https://cloudinary.com/documentation)
- [Cloudinary PHP SDK](https://github.com/cloudinary/cloudinary_php)
- [Cloudinary Image Transformations](https://cloudinary.com/documentation/image_transformations)
