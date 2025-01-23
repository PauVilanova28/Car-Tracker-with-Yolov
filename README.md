# Car Tracker
# Introduction  

This project focused on developing a **vehicle tracking system** to monitor and count vehicles passing near the engineering building, distinguishing their direction of movement (uphill or downhill). The system aimed to provide real-time traffic data, with an emphasis on efficiency and accuracy, using pre-recorded street videos to detect and track vehicles.

Key requirements included real-time frame processing to avoid delays or information loss.

# Objectives  

The primary goal was to create a **reliable tracking system** capable of:  
- Detecting and counting vehicles over varying time periods.  
- Determining movement direction (uphill or downhill).  
- Optimizing frame processing for real-time application.  
- Delivering accurate traffic monitoring data.

### Sub-Objectives  
1. **Data Preparation**: Simulate data capture using pre-recorded videos.  
2. **Vehicle Detection**: Develop or use an object detection model for vehicle identification.  
3. **Tracking Movement**: Implement tracking algorithms to monitor vehicle paths and direction.  
4. **Processing Optimization**: Ensure fast and efficient frame handling for real-time use.  
5. **Validation**: Evaluate system accuracy by comparing results with real-world data.

# Methodology  

The workflow involved four key steps:  

1. **Vehicle Detection**: Used pretrained models **YOLOv5n** and **YOLOv8n** to detect vehicles in video frames.  
   - **YOLOv8n**: Higher accuracy but slower.  
   - **YOLOv5n**: Faster with slightly reduced precision.  

2. **Tracking**: Leveraged the **DeepSORT tracker** with Kalman filters to maintain vehicle tracking across frames.  
   - Vehicles were assigned unique IDs for consistent tracking.  
   - Movement direction was determined using the bounding box’s `y-coordinate`.  

3. **Optimization**: Improved system performance by:  
   - **Image Cropping**: Focused on the area of interest to minimize unnecessary processing.  
   - **Frame Skipping**: Processed every 10th frame to reduce computational load while maintaining accuracy.  

4. **Validation**: Assessed system performance through:  
   - **Comparison with Provided Data**: Results were cross-checked against professor-supplied data.  
   - **Generated Video Analysis**: Verified tracking accuracy by reviewing system-generated videos.

# Experimental Design

This section outlines the process followed to create and select the most efficient model for vehicle tracking in video data. It discusses different detection model versions, detailing their selection criteria and key characteristics, as well as the tracking methods employed, focusing on the role of the DeepSORT tracker in ensuring continuous and accurate vehicle identification.

## Model 1 (Initial)

**Overview**: Model 1 uses a simple approach based on two reference lines placed at the start and end of the road. When a vehicle crosses a line, its position and direction are recorded based on the bounding box coordinates. Vehicles crossing the second line without changing direction are not recounted.

**Pros**:
- **Simplicity**: Easy to implement.
- **Basic Direction Tracking**: Identifies vehicle movement direction effectively.

**Cons**:
- **Inefficiency**: Frame-by-frame processing is computationally expensive.
- **False Detections**: Vehicles parked on the roadside or those on the upper road are mistakenly counted.  
- **Redundancy**: Performs unnecessary calculations for vehicles that have already been detected.

### Summary: 
Model 1 is a basic solution that works but lacks efficiency and accuracy, making it unsuitable for real-world applications.

## Model 2

**Overview**: Model 2 introduces a bounding box that isolates the target road, reducing false detections of parked vehicles or those on the upper road. Vehicles entering the bounding box are tracked for direction using movement between frames. The model processes one frame every 10 to enhance efficiency.

**Pros**:
- **Improved Efficiency**: Frame skipping accelerates processing.  
- **Reduced False Detections**: Bounding box helps ignore irrelevant vehicles.

**Cons**:
- **Recount Errors**: Some vehicles are counted multiple times.  
- **Parking Lot Interference**: Vehicles entering the nearby parking lot are still detected.
- **Direction Errors**: In some cases, movement direction is incorrectly calculated.

### Summary:
Model 2 improves speed and accuracy but still struggles with recounting and detecting vehicles entering unrelated areas.

## Model 3

**Overview**: Model 3 combines the strengths of previous models while introducing new enhancements. It uses a single reference line at the bottom of the frame to track vehicles exiting or entering the road. A margin is added around the reference line to account for frame-skipping and avoid missing vehicles. Additional criteria eliminate false detections of parked vehicles.

**Pros**:
- **High Performance**: Efficient frame skipping maintains processing speed.  
- **Accurate Movement Detection**: Properly tracks vehicles entering or exiting the road.  
- **Parked Vehicle Filtering**: Criteria eliminate vehicles mistakenly detected as moving.

**Cons**:
- **Edge Cases**: Vehicles parked close to the reference line may still be misclassified as moving.

### Summary:
Model 3 offers significant improvements in efficiency and accuracy, but minor errors remain in edge cases.

## Model 4 (Final)

**Overview**: Model 4 builds on Model 3 and introduces an additional detection line to eliminate errors caused by parked vehicles. Vehicles in contact with this new line for 10 consecutive frames are classified as stationary and excluded from the count.

**Pros**:
- **High Precision**: Efficiently filters out parked vehicles and tracks direction accurately.
- **Performance**: Maintains fast processing with frame skipping.  

**Cons**:
- None identified.  

### Summary:
Model 4 is the final, optimized version, combining the best features of previous models while addressing their limitations.

## Key Parameters of Model 4

### Tracking Criteria:
- **Minimum Movement Threshold**: Vehicles must move at least 48 pixels between 10 frames to be considered moving.  
- **Reference Line**: Located at `y=515` with a margin of 25 pixels to account for slight deviations.  
- **Stationary Frame Count**: Vehicles stationary for 10 frames are marked as parked.  
- **Movement Confirmation**: Vehicles must show movement over 4 frames to be counted.

### DeepSORT Parameters:
- **Max IoU Distance**: Set to 0.9 for precise object association.  
- **Max Age**: Vehicles can be undetected for up to 5 frames before being removed from tracking.  
- **Initialization Threshold**: Requires two detections to confirm a new vehicle.  

## Optimizations

Two strategies were explored to improve the system's performance:
1. **Using YOLOv5n**: A lighter version of YOLOv5 was tested, offering faster detection at the cost of reduced accuracy.  
2. **Image Cropping**: The image was cropped to focus solely on the target road, reducing computational load while maintaining detection accuracy.

These optimizations enhance the system's processing speed while preserving its precision, making it viable for practical use. The impact of these optimizations is analyzed in the results section.

# Results

This section presents a comparative analysis of the two most effective models, **Model 3** and **Model 4**, in terms of vehicle detection and counting accuracy. Additionally, it examines the impact of optimizations applied to the final version (Model 4), including image cropping and the use of YOLOv5n, a lighter detection model.

## 4.1 Comparison of Model 3 and Model 4

The comparison highlights key similarities and differences between the two models. While both demonstrate strong performance, Model 4 addresses specific limitations of Model 3, particularly in the **Long2** sequence, where Model 3 erroneously counts stationary vehicles near the reference line. Model 4 resolves this with an additional filtering mechanism that excludes parked cars, significantly improving accuracy.

### Results Overview:
| Sequence  | Count Up - Count Down (Model 3) | Real Up - Real Down | Processing Time (Model 3) | Processing Time (Real) |
|-----------|--------------------------------|----------------------|---------------------------|------------------------|
| Short     | 6 - 2                          | 6 - 2                | 00:02:49                 | 00:04:02              |
| Middle    | 5 - 7                          | 5 - 7                | 00:09:22                 | 00:07:00              |
| Shadow    | 3 - 10                         | 3 - 10               | 00:13:03                 | 00:12:03              |
| Long1     | 9 - 25                         | 8 - 24               | 00:37:19                 | 00:25:21              |
| Long2     | 8 - 134                        | 6 - 133              | 02:16:42                 | 01:04:54              |

| Sequence  | Count Up - Count Down (Model 4) | Real Up - Real Down | Processing Time (Model 4) | Processing Time (Real) |
|-----------|--------------------------------|----------------------|---------------------------|------------------------|
| Short     | 6 - 2                          | 6 - 2                | 00:02:42                 | 00:04:02              |
| Middle    | 5 - 7                          | 5 - 7                | 00:07:44                 | 00:07:00              |
| Shadow    | 3 - 10                         | 3 - 10               | 00:10:11                 | 00:12:03              |
| Long1     | 9 - 25                         | 8 - 24               | 00:21:38                 | 00:25:21              |
| Long2     | 7 - 133                        | 6 - 133              | 01:16:38                 | 01:04:54              |

### Key Observations:
1. **Improved Counting Accuracy**: Model 4 effectively eliminates errors caused by stationary vehicles near the reference line, particularly in the **Long2** sequence.
2. **Processing Time**: Model 4 demonstrates faster processing across all sequences compared to Model 3.

### Conclusion:
Model 4 outperforms Model 3 by providing more reliable vehicle counts, particularly in challenging sequences like Long2, and reduces processing time without compromising accuracy.

## 4.2 Impact of Optimizations on Model 4

The optimizations implemented in Model 4 include image cropping and the use of YOLOv5n to enhance efficiency. These adjustments aim to improve processing speed while maintaining high accuracy.

### Results with Image Cropping:
| Sequence  | Count Up - Count Down | Real Up - Real Down | Processing Time | Processing Time (Real) |
|-----------|------------------------|---------------------|----------------|------------------------|
| Short     | 6 - 2                  | 6 - 2               | 00:02:30       | 00:04:02              |
| Middle    | 5 - 7                  | 5 - 7               | 00:07:32       | 00:07:00              |
| Shadow    | 3 - 10                 | 3 - 10              | 00:09:19       | 00:12:03              |

### Results with YOLOv5n:
| Sequence  | Count Up - Count Down | Real Up - Real Down | Processing Time | Processing Time (Real) |
|-----------|------------------------|---------------------|----------------|------------------------|
| Short     | 5 - 2                  | 6 - 2               | 00:02:43       | 00:04:02              |
| Middle    | 5 - 7                  | 5 - 7               | 00:05:13       | 00:07:00              |
| Shadow    | 3 - 10                 | 3 - 10              | 00:07:43       | 00:12:03              |

### Observations:
1. **Image Cropping**:
   - Significantly reduces processing time without affecting accuracy.
   - Allows faster frame analysis while maintaining reliable results.

2. **YOLOv5n**:
   - Improves processing speed but occasionally undercounts vehicles, as seen in the **Short** sequence.
   - Adjusting the frame rate (e.g., processing 1 out of every 9 frames instead of 10) mitigates some errors but slightly increases processing time.

## Summary of Results:
- **Model 4**: Achieves robust accuracy and efficiency, with significant improvements over previous models.
- **Optimizations**: Image cropping enhances processing speed, while YOLOv5n offers a trade-off between speed and accuracy.

# Conclusions

The project successfully developed an efficient real-time vehicle tracking and counting system. The key takeaways and conclusions are as follows:

## 1. Overall System Performance
- **Model 4** proved to be the most effective version, delivering robust accuracy and efficiency in detecting and counting vehicles. It resolved key issues present in earlier models, such as miscounting stationary vehicles and processing delays.
- The system's accuracy was validated across multiple test sequences, achieving reliable results even in complex scenarios like the **Long2** sequence, where stationary vehicles previously caused miscounts.

## 2. Impact of Optimizations
- **Image Cropping**: Significantly reduced processing time without compromising accuracy. This method efficiently minimized the amount of data processed while maintaining reliable vehicle detection and counting.
- **YOLOv5n**: Provided faster processing but introduced occasional undercounting errors. Adjusting the frame processing rate from 1 frame per 10 to 1 frame per 9 improved detection accuracy, though it slightly increased processing time.

## 3. Improvements in Accuracy and Efficiency
- Splitting the tracking process into distinct components, such as vehicle movement direction and stationary vehicle filtering, significantly enhanced system performance.
- The addition of a secondary line for detecting stationary vehicles effectively reduced errors, making the system suitable for real-world applications.

## 4. Limitations
- Despite the improvements, the system occasionally struggled with specific edge cases, such as high-speed vehicles or vehicles closely aligned with the reference lines.
- YOLOv5n, while faster, introduced some inconsistencies in detection, requiring further fine-tuning for optimal results.

## 5. Future Work
To further enhance the system, future work could include:
- Exploring lighter object detection models to improve processing speed while maintaining accuracy.
- Incorporating additional tracking and filtering techniques to handle high-speed or closely aligned vehicles.
- Extending the system to support diverse traffic scenarios, such as different road types or multi-lane environments.
- Developing adaptive models that can dynamically adjust detection thresholds based on real-time conditions.

## Final Remarks
The final version of the system demonstrates robust accuracy and processing efficiency, with significant potential for real-world applications in traffic monitoring. By addressing the limitations and implementing suggested future improvements, this system could become a highly reliable tool for urban traffic analysis.

