# PhotoEditorApp
To create a **Photo Editor App** in Android Studio, you can use popular image editing libraries that provide powerful tools to manipulate photos. Here's a step-by-step guide using a common library like **PhotoEditor SDK** or **uCrop** for cropping and basic editing features.

### 1. **Setup Android Project**
- Open **Android Studio** and create a new project with an empty activity.
- Choose a project name, package name, and language (Kotlin or Java).

### 2. **Add Required Dependencies**

For a **Photo Editor SDK** or **uCrop**, you need to include the required libraries in your `build.gradle` file.

- **PhotoEditor SDK** (an open-source alternative like `PhotoView` for zooming, or `image-cropper` for cropping):
  ```gradle
  implementation 'ja.burhanrashid52:photoeditor:1.1.0'
  ```

- **uCrop** (for cropping images):
  ```gradle
  implementation 'com.github.yalantis:ucrop:2.2.8'
  ```

Sync your project after adding these dependencies.

### 3. **Design the Layout**
You can create a simple layout with an ImageView to display the photo, buttons for editing tools (crop, filter, rotate), and options to save or reset the photo.

Here's a basic `activity_main.xml` layout:

```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <ImageView
        android:id="@+id/photoView"
        android:layout_width="match_parent"
        android:layout_height="400dp"
        android:scaleType="fitCenter"
        android:contentDescription="Photo Preview" />

    <Button
        android:id="@+id/btnEdit"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Edit" />

    <Button
        android:id="@+id/btnCrop"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Crop" />

    <Button
        android:id="@+id/btnSave"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Save" />

</LinearLayout>
```

### 4. **Initialize the Photo Editor**
In your `MainActivity.kt` or `MainActivity.java`, initialize the PhotoEditor and handle basic operations such as crop, rotate, and apply filters.

Here’s an example in Kotlin:

```kotlin
class MainActivity : AppCompatActivity() {
    lateinit var photoEditor: PhotoEditor
    lateinit var photoView: ImageView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        photoView = findViewById(R.id.photoView)

        val btnEdit = findViewById<Button>(R.id.btnEdit)
        val btnCrop = findViewById<Button>(R.id.btnCrop)
        val btnSave = findViewById<Button>(R.id.btnSave)

        // Set a sample image to edit
        val bitmap = BitmapFactory.decodeResource(resources, R.drawable.sample_photo)
        photoView.setImageBitmap(bitmap)

        // Initialize the PhotoEditor
        val photoEditorView = PhotoEditorView(this)
        photoEditor = PhotoEditor.Builder(this, photoEditorView)
            .setPinchTextScalable(true)
            .build()

        // Edit button to add text, stickers, etc.
        btnEdit.setOnClickListener {
            // Add text example
            photoEditor.addText("Sample Text", Color.RED)
        }

        // Crop button to open uCrop activity
        btnCrop.setOnClickListener {
            startCropActivity(Uri.parse("file://path/to/image"))
        }

        // Save button
        btnSave.setOnClickListener {
            saveImage()
        }
    }

    private fun startCropActivity(imageUri: Uri) {
        // Launch UCrop for cropping
        val destinationUri = Uri.fromFile(File(cacheDir, "cropped_image.jpg"))
        UCrop.of(imageUri, destinationUri)
            .withAspectRatio(16F, 9F)
            .withMaxResultSize(1000, 1000)
            .start(this)
    }

    private fun saveImage() {
        // Save edited image
        photoEditor.saveAsFile("/path/to/save/image.jpg", object : PhotoEditor.OnSaveListener {
            override fun onSuccess(imagePath: String) {
                Toast.makeText(this@MainActivity, "Image Saved: $imagePath", Toast.LENGTH_SHORT).show()
            }

            override fun onFailure(exception: Exception) {
                Toast.makeText(this@MainActivity, "Save Failed", Toast.LENGTH_SHORT).show()
            }
        })
    }

    // Handle the result from UCrop (cropping)
    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)
        if (resultCode == RESULT_OK && requestCode == UCrop.REQUEST_CROP) {
            val resultUri = UCrop.getOutput(data!!)
            photoView.setImageURI(resultUri)  // Set the cropped image
        }
    }
}
```

### 5. **Cropping Images with uCrop**
For cropping, `uCrop` provides a straightforward way to allow users to adjust their images before saving.

In the above code, `startCropActivity()` opens the uCrop activity where users can crop the image. You can customize the aspect ratio, result size, and more.

### 6. **Adding Filters and Text**
With the **PhotoEditor SDK**, you can add text overlays, filters, and stickers to images.

Example of adding text:
```kotlin
photoEditor.addText("Hello World", Color.RED)
```

To add filters, you can use the built-in filters from the PhotoEditor SDK:
```kotlin
photoEditor.setFilterEffect(PhotoFilter.BLACK_WHITE)
```

### 7. **Saving the Edited Image**
Once the user has finished editing, you can save the image either to the device storage or share it using Android's **Intent.ACTION_SEND**:

```kotlin
val filePath = "/storage/emulated/0/Pictures/edited_photo.jpg"
photoEditor.saveAsFile(filePath, object : PhotoEditor.OnSaveListener {
    override fun onSuccess(imagePath: String) {
        Toast.makeText(this@MainActivity, "Image saved to $imagePath", Toast.LENGTH_SHORT).show()
    }

    override fun onFailure(exception: Exception) {
        Toast.makeText(this@MainActivity, "Failed to save image", Toast.LENGTH_SHORT).show()
    }
})
```

### 8. **Additional Features**
- **Stickers:** You can add sticker images to the photo.
  ```kotlin
  photoEditor.addImage(BitmapFactory.decodeResource(resources, R.drawable.sticker))
  ```

- **Brush Tool:** Allow users to draw on the photo using a brush tool.
  ```kotlin
  photoEditor.setBrushDrawingMode(true)
  photoEditor.brushColor = Color.YELLOW
  ```

### 9. **UI Enhancements**
- Add more buttons or options for editing features like brightness, contrast, rotation, etc.
- Use custom icons for crop, save, and other editing features for better user experience.

### 10. **Test and Optimize**
- Test the app on different devices and orientations.
- Optimize performance by managing bitmap memory usage to avoid **OutOfMemoryException**.
