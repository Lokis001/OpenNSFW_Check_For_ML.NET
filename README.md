# OpenNSFW for ML.NET

[![.NET](https://img.shields.io/badge/.NET-8.0%2B-blueviolet)](https://dotnet.microsoft.com/)
[![ML.NET](https://img.shields.io/badge/ML.NET-4.0%2B-red)](https://dotnet.microsoft.com/apps/machinelearning-ai/ml-dotnet)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)
[![Dataset Size](https://img.shields.io/badge/Dataset-7.21GB-orange)](https://github.com/yourusername/OpenNSFW_Check_For_ML.NET)
[![Images Count](https://img.shields.io/badge/Images-6107-blue)](https://github.com/yourusername/OpenNSFW_Check_For_ML.NET)
[![Dataset Version](https://img.shields.io/badge/Version-1.0.1r0-yellow)](https://github.com/Lokis001/OpenNSFW_Check_For_ML.NET)

A pre-trained model for classifying images into NSFW (Not Safe For Work) and SFW (Safe For Work) categories, specifically designed for use with ML.NET in C# applications.

## 📋 Table of Contents

- [Features](#features)
- [Technical Details](#technical-details)
- [Quick Start](#quick-start)
- [Code Usage](#code-usage)
- [Dataset Information](#dataset-information)
- [Usage Examples](#usage-examples)
- [Limitations and Warnings](#limitations-and-warnings)
- [Installation](#installation)
- [License](#license)
- [Support](#support)

## ✨ Features

- **Ready-to-use ML.NET model** - No additional training required
- **Easy integration** - Simple to add to existing C# projects
- **High performance** - Optimized for .NET ecosystem
- **Binary classification** - NSFW (18+) and SFW (safe) content
- **Popular format support** - JPG, PNG, BMP, GIF
- **Regular updates** - Dataset expands every 2,000 new images
- **Version tracking** - Clear versioning for model and dataset

## 🔧 Technical Details

- **Machine Learning Framework**: ML.NET 4.0+
- **Model Type**: TensorFlow model converted for ML.NET
- **Output**: Probabilities for two classes (SFW/NSFW)
- **Current Model Version**: 1.0.1r0
- **Dataset Version**: 1.0.1r0

## 🚀 Quick Start

### Prerequisites

- [.NET SDK](https://dotnet.microsoft.com/download)
- [Visual Studio 2022](https://visualstudio.microsoft.com/) or [VS Code](https://code.visualstudio.com/)
- NuGet package: `Microsoft.ML`

## 📊 Dataset Information

### Current Version: 1.0.1r0

| Parameter | Value |
|-----------|-------|
| Total images | 6,107 |
| Dataset size | 7.21 GB |
| NSFW class images | ~2,683 |
| SFW class images | ~3,424 |
| Image formats | JPG, PNG, GIF |
| Data sources | Manual moderation |

### 📈 Update Schedule

The dataset follows a structured expansion plan:

- **Regular updates**: Every 2,000 new images added
- **Version increment**: Minor version updates with each expansion
- **Quality control**: Manual verification of new additions
- **Balanced growth**: Maintains ratio between SFW/NSFW classes

#### Planned Expansion Roadmap:
- **v1.0.1r0** (Current): 6,107 images ✓
- **v1.1.0**: ~8,000 images (planned)


### 🔄 Update Process

```csharp
public class DatasetVersionInfo
{
    // Current version
    public string Version => "1.0.1r0";
    public int TotalImages => 6107;
    public DateTime LastUpdate => new DateTime(2024, 1, 15);
    
    // Update tracking
    public bool CheckForUpdates()
    {
        // Implementation for checking new dataset versions
        return false; // Placeholder
    }
    
    public void DownloadUpdate(string newVersion)
    {
        // Implementation for downloading dataset updates
        Console.WriteLine($"Downloading dataset update to {newVersion}");
    }
}
```

## 💻 Code Usage

### Basic Implementation

```csharp
using Microsoft.ML;
using System;
using System.IO;

namespace NSFW_Detection
{
    class Program
    {
        static void Main(string[] args)
        {
            // Display version information
            Console.WriteLine($"OpenNSFW for ML.NET - Version 1.0.1r0");
            Console.WriteLine($"Dataset: 6,107 images (7.21 GB)");
            Console.WriteLine($"Last updated: January 15, 2024\n");
            
            // Initialize MLContext
            var mlContext = new MLContext();
            
            // Load trained model (version-specific)
            var modelPath = $"OpenNSFWModel_v1.0.1r0.mlmodel";
            var model = mlContext.Model.Load(modelPath, out var modelSchema);
            
            // Create prediction engine
            var predictionEngine = mlContext.Model.CreatePredictionEngine<ModelInput, ModelOutput>(model);
            
            // Example usage
            var input = new ModelInput
            {
                ImagePath = "path/to/your/image.jpg"
            };
            
            // Make prediction
            var prediction = predictionEngine.Predict(input);
            
            // Output results
            Console.WriteLine($"SFW Probability: {prediction.SFWScore:P2}");
            Console.WriteLine($"NSFW Probability: {prediction.NSFWScore:P2}");
            Console.WriteLine($"Classification: {(prediction.IsNSFW ? "NSFW (18+)" : "SFW (Safe)")}");
            Console.WriteLine($"\nModel confidence based on 6,107 training images");
        }
    }
    
    // Model classes with version info
    public class ModelInput
    {
        public string ImagePath { get; set; }
        public string ModelVersion => "1.0.1r0"; // Embedded version info
    }
    
    public class ModelOutput
    {
        public float[] Score { get; set; }
        public float SFWScore => Score[0];
        public float NSFWScore => Score[1];
        public bool IsNSFW => NSFWScore > 0.5f;
        public string Version => "1.0.1r0";
    }
}
```

### Advanced Usage with Version Awareness

```csharp
using Microsoft.ML;
using System;
using System.Collections.Generic;
using System.IO;

public class VersionedNSFWDetector
{
    private readonly PredictionEngine<ModelInput, ModelOutput> _predictionEngine;
    private readonly MLContext _mlContext;
    private readonly string _modelVersion;
    
    public VersionedNSFWDetector(string modelPath)
    {
        _mlContext = new MLContext();
        var model = _mlContext.Model.Load(modelPath, out _);
        _predictionEngine = _mlContext.Model.CreatePredictionEngine<ModelInput, ModelOutput>(model);
        _modelVersion = ExtractVersionFromPath(modelPath);
    }
    
    public string ModelVersion => _modelVersion;
    
    public DetectionResult AnalyzeImage(string imagePath, float threshold = 0.75f)
    {
        if (!File.Exists(imagePath))
            throw new FileNotFoundException($"Image not found: {imagePath}");
        
        var input = new ModelInput { ImagePath = imagePath };
        var prediction = _predictionEngine.Predict(input);
        
        return new DetectionResult
        {
            ImagePath = imagePath,
            SFWConfidence = prediction.SFWScore,
            NSFWConfidence = prediction.NSFWScore,
            IsNSFW = prediction.NSFWScore >= threshold,
            IsSafe = prediction.SFWScore >= threshold,
            IsAmbiguous = Math.Abs(prediction.SFWScore - prediction.NSFWScore) < 0.2,
            ModelVersion = _modelVersion,
            TrainingSetSize = GetTrainingSetSizeForVersion(_modelVersion)
        };
    }
    
    private string ExtractVersionFromPath(string path)
    {
        // Extract version from filename pattern: OpenNSFWModel_vX.X.XrX.zip
        var filename = Path.GetFileNameWithoutExtension(path);
        if (filename.Contains("_v"))
        {
            return filename.Split(new[] { "_v" }, StringSplitOptions.None)[1];
        }
        return "1.0.1r0"; // Default to current version
    }
    
    private int GetTrainingSetSizeForVersion(string version)
    {
        // Map versions to dataset sizes
        var versionMap = new Dictionary<string, int>
        {
            ["1.0.1r0"] = 6107
            // Add new versions here as dataset expands
        };
        
        return versionMap.TryGetValue(version, out var size) ? size : 6107;
    }
}

public class DetectionResult
{
    public string ImagePath { get; set; }
    public float SFWConfidence { get; set; }
    public float NSFWConfidence { get; set; }
    public bool IsNSFW { get; set; }
    public bool IsSafe { get; set; }
    public bool IsAmbiguous { get; set; }
    public string ModelVersion { get; set; }
    public int TrainingSetSize { get; set; }
    
    public override string ToString()
    {
        return $"{(IsNSFW ? "NSFW" : "SFW")} (Confidence: {Math.Max(SFWConfidence, NSFWConfidence):P0}) " +
               $"[Model v{ModelVersion}, trained on {TrainingSetSize:N0} images]";
    }
}
```

## 🔒 Limitations and Warnings

### ⚠️ Important Warnings

1. **Model accuracy**: ~85-90% on test data (based on 6,107 images)
2. **False positives/negatives**: Both false positives and false negatives are possible
3. **Context**: Model analyzes only visual content, doesn't understand context
4. **Age restrictions**: Model is trained to recognize adult content but is not perfect

### Dataset-Specific Notes

- **Current coverage**: 6,107 manually verified images
- **Expanding dataset**: Regular additions every 2,000 images
- **Version updates**: Accuracy expected to improve with dataset growth
- **Retraining schedule**: Models retrained after significant dataset expansions

### Usage Recommendations

- Use the model as an **additional filter**, not the sole criterion
- For critical applications, add manual moderation
- Set appropriate confidence thresholds for your use case
- Monitor for new versions with expanded training data
- Consider retraining when major dataset updates occur (every ~6,000 images)

## 📈 Usage Examples

### Web Application with Version Awareness

```csharp
// Integration with ASP.NET Core
public class ContentModerationService
{
    private readonly VersionedNSFWDetector _detector;
    
    public ContentModerationService()
    {
        _detector = new VersionedNSFWDetector("OpenNSFWModel_v1.0.1r0.zip");
        
        // Log version information
        Console.WriteLine($"Initialized NSFW Detector v{_detector.ModelVersion}");
        Console.WriteLine($"Training dataset: 6,107 images");
    }
    
    public async Task<ModerationResult> ModerateUserUpload(IFormFile imageFile)
    {
        // Save temporary file
        var tempPath = Path.GetTempFileName();
        using (var stream = new FileStream(tempPath, FileMode.Create))
        {
            await imageFile.CopyToAsync(stream);
        }
        
        // Analyze image
        var result = _detector.AnalyzeImage(tempPath);
        
        // Cleanup
        File.Delete(tempPath);
        
        return new ModerationResult
        {
            IsApproved = !result.IsNSFW,
            Confidence = result.NSFWConfidence,
            RequiresManualReview = result.IsAmbiguous,
            ModelVersion = result.ModelVersion,
            TrainingSetSize = result.TrainingSetSize,
            Metadata = $"Based on {result.TrainingSetSize:N0} training images"
        };
    }
}
```

### Batch Processing with Progress Tracking

```csharp
public class BatchProcessor
{
    public void ProcessWithProgress(string inputDir, string outputDir)
    {
        var detector = new VersionedNSFWDetector("OpenNSFWModel_v1.0.1r0.zip");
        var imageFiles = Directory.GetFiles(inputDir, "*.*")
            .Where(f => f.EndsWith(".jpg") || f.EndsWith(".png") || f.EndsWith(".bmp"));
        
        int processed = 0;
        int nsfwCount = 0;
        
        Console.WriteLine($"Starting batch processing with model v{detector.ModelVersion}");
        Console.WriteLine($"Training dataset size: 6,107 images");
        Console.WriteLine($"Processing {imageFiles.Count()} images...\n");
        
        foreach (var imageFile in imageFiles)
        {
            processed++;
            var result = detector.AnalyzeImage(imageFile);
            
            if (result.IsNSFW)
                nsfwCount++;
            
            // Display progress
            if (processed % 100 == 0)
            {
                Console.WriteLine($"Processed {processed}/{imageFiles.Count()} images...");
                Console.WriteLine($"Current NSFW rate: {(double)nsfwCount/processed:P1}");
            }
        }
        
        Console.WriteLine($"\nProcessing complete!");
        Console.WriteLine($"Total processed: {processed}");
        Console.WriteLine($"NSFW detected: {nsfwCount} ({(double)nsfwCount/processed:P1})");
        Console.WriteLine($"Model version: {detector.ModelVersion}");
        Console.WriteLine($"\nNote: Model trained on 6,107 images. " +
                         $"Dataset expands every 2,000 new images.");
    }
}
```

## 📦 NuGet Installation

Add to your `.csproj` file:

```xml
<ItemGroup>
  <PackageReference Include="Microsoft.ML" Version="4.0.0" />
  <PackageReference Include="Microsoft.ML.Vision" Version="4.0.0" />
  <PackageReference Include="SciSharp.TensorFlow.Redist" Version="2.3.1" />
</ItemGroup>
```

## 📄 License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## 🔄 Update Notifications

To stay informed about dataset expansions and new versions:

1. **Watch the repository** for new releases
2. **Check version badges** in this README
3. **Monitor dataset size** (grows by 2,000 images periodically)
4. **Review CHANGELOG.md** for version history

### Version History

| Version | Images | Size | Release Date | Notes |
|---------|--------|------|--------------|-------|
| 1.0.1r0 | 6,107 | 7.21 GB | Jan 15, 2024 | Initial release |
| Future | +2,000 | ~+2.4 GB | TBD | Planned expansion |

## 📞 Support

If you have questions or suggestions:

1. Open an [Issue](https://github.com/Lokis001/OpenNSFW_Check_For_ML.NET/issues)
2. Refer to ML.NET documentation
3. Check for dataset update announcements

---

**Note**: This model is intended to assist with content moderation and should not be the sole criterion for content blocking decisions. Always use human moderation for important decisions.

**Dataset Expansion**: The training dataset is actively growing and receives approximately 2,000 new manually verified images with each update, improving model accuracy over time. Current version: **1.0.1r0** (6,107 images). If you want to help with the development of this project, you can write to me on Telegram **@Alan_Thread**.
