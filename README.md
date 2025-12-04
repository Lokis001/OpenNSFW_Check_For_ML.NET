# OpenNSFW_Check for ML.NET

[![.NET](https://img.shields.io/badge/.NET-8.0%2B-blueviolet)](https://dotnet.microsoft.com/)
[![ML.NET](https://img.shields.io/badge/ML.NET-4.0%2B-red)](https://dotnet.microsoft.com/apps/machinelearning-ai/ml-dotnet)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)
[![Dataset Size](https://img.shields.io/badge/Dataset-7.21GB-orange)](https://github.com/yourusername/OpenNSFW_Check_For_ML.NET)
[![Images Count](https://img.shields.io/badge/Images-6107-blue)](https://github.com/yourusername/OpenNSFW_Check_For_ML.NET)

A pre-trained model for classifying images into NSFW (Not Safe For Work) and SFW (Safe For Work) categories, specifically designed for use with ML.NET in C# applications.

## 📋 Table of Contents

- [Features](#features)
- [Technical Details](#technical-details)
- [Quick Start](#quick-start)
- [Code Usage](#code-usage)
- [Dataset Metadata](#dataset-metadata)
- [Usage Examples](#usage-examples)
- [Limitations and Warnings](#limitations-and-warnings)
- [Installation](#installation)
- [License](#license)

## ✨ Features

- **Ready-to-use ML.NET model** - No additional training required
- **Easy integration** - Simple to add to existing C# projects
- **High performance** - Optimized for .NET ecosystem
- **Binary classification** - NSFW (18+) and SFW (safe) content
- **Popular format support** - JPG, PNG, BMP, GIF

## 🔧 Technical Details

- **Machine Learning Framework**: ML.NET 4.0+
- **Model Type**: TensorFlow model converted for ML.NET
- **Output**: Probabilities for two classes (SFW/NSFW)

## 🚀 Quick Start

### Prerequisites

- [.NET SDK](https://dotnet.microsoft.com/download)
- [Visual Studio 2022](https://visualstudio.microsoft.com/) or [VS Code](https://code.visualstudio.com/)
- NuGet package: `Microsoft.ML`

## 📊 Dataset Metadata

| Parameter | Value |
|-----------|-------|
| Total images | 6,107 |
| Dataset size | 7.21 GB |
| NSFW class images | ~2,683 |
| SFW class images | ~3,424 |
| Image formats | JPG, PNG, GIF |
| Data sources | Manual moderation |

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
            // Initialize MLContext
            var mlContext = new MLContext();
            
            // Load trained model
            var model = mlContext.Model.Load("OpenNSFWModel.zip", out var modelSchema);
            
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
        }
    }
    
    // Model classes
    public class ModelInput
    {
        public string ImagePath { get; set; }
    }
    
    public class ModelOutput
    {
        public float[] Score { get; set; }
        public float SFWScore => Score[0];
        public float NSFWScore => Score[1];
        public bool IsNSFW => NSFWScore > 0.5f;
    }
}
```

### Advanced Usage

```csharp
using Microsoft.ML;
using Microsoft.ML.Data;
using Microsoft.ML.Transforms;
using System;
using System.Collections.Generic;
using System.Drawing;
using System.IO;
using System.Linq;

public class AdvancedNSFWDetector
{
    private readonly PredictionEngine<ModelInput, ModelOutput> _predictionEngine;
    private readonly MLContext _mlContext;
    
    public AdvancedNSFWDetector(string modelPath)
    {
        _mlContext = new MLContext();
        var model = _mlContext.Model.Load(modelPath, out _);
        _predictionEngine = _mlContext.Model.CreatePredictionEngine<ModelInput, ModelOutput>(model);
    }
    
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
            IsAmbiguous = Math.Abs(prediction.SFWScore - prediction.NSFWScore) < 0.2
        };
    }
    
    public BatchDetectionResult AnalyzeDirectory(string directoryPath, float threshold = 0.75f)
    {
        var imageFiles = Directory.GetFiles(directoryPath, "*.jpg")
            .Concat(Directory.GetFiles(directoryPath, "*.png"))
            .Concat(Directory.GetFiles(directoryPath, "*.bmp"));
        
        var results = new List<DetectionResult>();
        int nsfwCount = 0;
        
        foreach (var imageFile in imageFiles)
        {
            var result = AnalyzeImage(imageFile, threshold);
            results.Add(result);
            
            if (result.IsNSFW)
                nsfwCount++;
        }
        
        return new BatchDetectionResult
        {
            TotalImages = results.Count,
            NSFWCount = nsfwCount,
            SFWCount = results.Count - nsfwCount,
            Results = results
        };
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
}

public class BatchDetectionResult
{
    public int TotalImages { get; set; }
    public int NSFWCount { get; set; }
    public int SFWCount { get; set; }
    public List<DetectionResult> Results { get; set; }
}
```

## 🔒 Limitations and Warnings

### ⚠️ Important Warnings

1. **Model accuracy**: ~85-90% on test data
2. **False positives/negatives**: Both false positives and false negatives are possible
3. **Context**: Model analyzes only visual content, doesn't understand context
4. **Age restrictions**: Model is trained to recognize adult content but is not perfect

### Usage Recommendations

- Use the model as an **additional filter**, not the sole criterion
- For critical applications, add manual moderation
- Set appropriate confidence thresholds for your use case
- Regularly update the model for improved accuracy

## 📈 Usage Examples

### Web Application for Content Moderation
```csharp
// Integration with ASP.NET Core
public class ContentModerationService
{
    public async Task<ModerationResult> ModerateUserUpload(IFormFile imageFile)
    {
        // Save temporary file
        var tempPath = Path.GetTempFileName();
        using (var stream = new FileStream(tempPath, FileMode.Create))
        {
            await imageFile.CopyToAsync(stream);
        }
        
        // Analyze image
        var detector = new AdvancedNSFWDetector("OpenNSFWModel.zip");
        var result = detector.AnalyzeImage(tempPath);
        
        // Cleanup
        File.Delete(tempPath);
        
        return new ModerationResult
        {
            IsApproved = !result.IsNSFW,
            Confidence = result.NSFWConfidence,
            RequiresManualReview = result.IsAmbiguous
        };
    }
}
```

### Batch Image Processing
```csharp
// Batch processing of image directory
public void ProcessImageDirectory(string inputDir, string safeDir, string nsfwDir)
{
    var detector = new AdvancedNSFWDetector("OpenNSFWModel.zip");
    var batchResult = detector.AnalyzeDirectory(inputDir);
    
    foreach (var result in batchResult.Results)
    {
        var destinationDir = result.IsNSFW ? nsfwDir : safeDir;
        var fileName = Path.GetFileName(result.ImagePath);
        var destinationPath = Path.Combine(destinationDir, fileName);
        
        File.Copy(result.ImagePath, destinationPath, true);
        
        Console.WriteLine($"Processed: {fileName} - " +
                         $"{(result.IsNSFW ? "NSFW" : "SFW")} " +
                         $"(Confidence: {Math.Max(result.SFWConfidence, result.NSFWConfidence):P0})");
    }
    
    Console.WriteLine($"\nSummary:");
    Console.WriteLine($"Total processed: {batchResult.TotalImages}");
    Console.WriteLine($"SFW: {batchResult.SFWCount}");
    Console.WriteLine($"NSFW: {batchResult.NSFWCount}");
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


## 📞 Support

If you have questions or suggestions:

1. Open an [Issue](https://github.com/Lokis001/OpenNSFW_Check_For_ML.NET/issues)
2. Refer to ML.NET documentation

---

**Note**: This model is intended to assist with content moderation and should not be the sole criterion for content blocking decisions. Always use human moderation for important decisions.
