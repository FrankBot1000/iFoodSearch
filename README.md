# iFoodSearch
iFoodSearch is a feature rich iOS App built using Swift and UIKit. iFoodSearch is organized into five tab view screens: Search, Favorites, Diary, Charts and Settings. 

You can search for food or recipe nutrient data, add food favorites, create a food diary, track diary nutrient data in Charts, and provides many customization features (in Settings).
<br></br>

<h2>Available on the iOS AppStore:</h2>
<a href="https://apps.apple.com/us/app/ifoodsearch/id6467825358">
	<img src="images/pics/Download_on_the_App_Store_Badge_US-UK_RGB_blk_092917.svg" alt="Download on App Store">
</a>
<a href="https://www.apple.com/ca/ios/health/">
	<img src="images/pics/Apple_Health_badge_US-UK_blk_sRGB.svg" alt="Works with Apple Health">
</a>
<br></br>

# Technologies Used		
## Languages and Frameworks						
* Swift Programming language
* Assembly Language
* Apple's AppKit framework

## Apple Technologies
* Core Data
* Core Graphics
* StoreKit
* AutoLayout
* Unit Testing
* Documentation (DocC)
* On-Demand Resources

## External APIs
* USDA FoodCentral
* FatSecret Swift
* iOS SecuritySuite
* OpenSSL

## Other
* Git Version Control
<br></br>


# iFoodSearch Animations
[//]: # "NB: For README.md Github videos, Use GitHub asset urls eg. https://github.com/user-attachments/assets/xxxxxPlaceholderFileNameHerexxxxx as video source (derived first by dragging-dropping a video within the README.md file to get the url)."
<!-- 
<video width="500" src="https://github.com/user-attachments/assets/xxxxxPlaceholderFileNameHerexxxxx">
</video>
<video width="500" src="https://github.com/user-attachments/assets/xxxxxPlaceholderFileNameHerexxxxx">
</video>
 -->

[//]: # "For webpage, use embedded below figure instead."
<div style="display: inline-block">
	<figure>
		<div style="display: block; position: absolute; z-index: 2; pointer-events: none;">
			<img style="width: 210px;" src="images/pics/iPhone 14 Pro - Space Black - Portrait.png" alt="iPhone Pic1" />
		</div>
		<div style="display: inline-block; position: relative; z-index: 1;">
			<video style="margin: 12px; width: 186px; border-radius: 15px;" class="iphonevideo" controls poster="images/pics/Food_Info_Details_Potatoes_ABS_copyright_FBotLogic_Solutions_Inc.png" muted preload="auto">
				<source src="videos/01a iFoodSearch potato food info.mp4" type="video/mp4">
				<!- - For non-HTML5 browsers: - ->
				Your browser doesn't support the video tag. Click <a href=http://www.firefox.com>here</a> 
				to download the Firefox browser for your operating system.
			</video>
		</div>
	</figure>
</div>

<div style="display: inline-block">
	<figure>
		<div style="display: block; position: absolute; z-index: 2; pointer-events: none;">
			<img style="width: 210px;" src="images/pics/iPhone 14 Pro - Space Black - Portrait.png" alt="iPhone Pic1" />
		</div>
		<div style="display: inline-block; position: relative; z-index: 1;">
			<video style="margin: 12px; width: 186px; border-radius: 15px;" class="iphonevideo" controls poster="images/pics/iPhone15Pro_Food_Time_Chart_copyright_FBotLogic_Solutions_Inc.png" muted preload="auto">
				<source src="videos/01b iFoodSearch diary chart info.mp4" type="video/mp4">
				<!- - For non-HTML5 browsers: - ->
				Your browser doesn't support the video tag. Click <a href=http://www.firefox.com>here</a> 
				to download the Firefox browser for your operating system.
			</video>
		</div>
	</figure>
</div>
<br></br>


# A Full Featured iOS Nutrient Tracker App...
iFoodSearch started as a simple food and recipe search App for finding nutrient information. I extended the <a href="https://github.com/NicholasBellucci/FatSecretSwift">FatSecret Swift API</a> by Nicolas Bellucci, that only searched for food-specific data, to include recipe searches. You can find it <a href="https://github.com/FrankBot1000/FatSecretSwift">here</a>.

Yet, nothing is simple, if motivated enough... iFoodSearch now also calculates food diary totals, syncs with Apple's Health App, and shows nutrient details in multiple chart types. And, it's highly customizable.

There have been many "show-stoppers" (i.e. challenges) if not for stubborn persistence. These included integrating HealthKit, adding security features like image validation (I wrote it in Assembly!), implementing Core Data models using Generics, building Custom Charts (for older iOS compatibility), implementing Certificate Pinning, and everyone's pain point, implementing StoreKit (with receipt validation). 
<br></br>


## Below are Some Code Examples:
### Testing HealthKit Sample Types:
```swift
    func testGetHKSampleTypesWithFDRFoodUUID_checkIfHKSampleTypesArrayIsNotEmpty() {
        // Given
        guard let fdrFood   = kMockData.fdrFoodsFunDiaryFoods.first else {
            print("fdrFood is nil!!!")
            return
        }
        guard let fdrFoodUUID   = fdrFood.uuid else { return }
        let date: Date = fdrFood.date!   // Changing date to accomodate data written the day before.
        
        // When
        let hkSampleTypes: [HKQuantitySample] = {
            FBHealthKit.fdrFoodNut_HKTypeID_Dict.keys.compactMap({ hkTypeID_Dict_key in
                let hkType = HKSampleType.quantityType(forIdentifier: hkTypeID_Dict_key)
                let hkTypeUUID = fdrFoodUUID + (FBHealthKit.fdrFoodNut_HKTypeID_Dict[hkTypeID_Dict_key]!).stringValue
                var hkSampleType: HKQuantitySample?
                let asyncHealthKitExpectation = self.expectation(description: "asyncHealthKitExpectation")
                FBHealthKit.getHKSampleType(hkType!, with: hkTypeUUID, date: date) { (sample, error) in
                    guard let sample = sample else {
                        if let error = error {
                            print("There was an error retreiving sample type: \(error)")
                        }
                        asyncHealthKitExpectation.fulfill()
                        return
                    }
                    asyncHealthKitExpectation.fulfill()
                    hkSampleType = sample
                    return
                }
                self.wait(for: [asyncHealthKitExpectation], timeout: 10)
                return hkSampleType
            })
        }()
        
        // Then
        XCTAssert(hkSampleTypes.isEmpty == false, "hkSampleTypes IS EMPTY.")
    }
```
<br></br>


### Validating image assets by comparing image bytes to expected bytes, in Assembly!
```Assembly
_checkbyte:
    ldrh w7, [x9, #-2]!     // doing a 2-byte step decrement; start by subtracting 2-bytes from X9, to get to start of last byte-pair, then write back resultant address back into X9.
    ldrh w8, [x10, #-2]!    // ...will start with comparing end of "Block #2", at X9, with end of "Block #1", at X10.
                            // ...NB: 'ldrh' loads half-word, 16bits or 2-bytes into W7 or W8.
    
    asr w7, w7, #8          // shift high byte of byte-pair into lowest byte (is where image data is), relevant data for checking image is at 2nd position in .data (in little endian format)
    asr w8, w8, #8
//    and w7, w7, #0xFF       // no longer necessary, since started with only 16-bits by using "h" suffix with 'ldr' ....clear all bits except lowest byte
//    and w8, w8, #0xFF
    eor w7, w7, w8          // w7 ^ w8 = "image byte", result stored in w7
    
    ldrb w6, [x2, #-4]!      // doing a 4-byte step decrement for Arg1 (recall: Arg1 address points to a 32 item array of 32bit (4-byte) values
//    and x6, x6, #0xFF     // no longer necessary, ...since only loaded lowest byte into 'w6'
    eor w6, w6, w7          // test equivalence, check if x7 = x6 (the relevant "image byte",
                            // i.e., if bits are the same, will be zero'd ('teq' instruction checks only a single bit)
    cmp w6, #0              // ...clang compiler doesn't accept 's' suffix for 'eor' for saving flags, so include "cmp x6, #0"
    b.eq _continue          // branch if not equal

    orr x0, x6, x7          // ...since x6 and x7 are not the same, ORR them together and return a non-zero value in X0 return register; to indicate that somethings wrong!
    add sp, sp, #16
    mov sp, fp
    ldp lr, fp, [sp], #16
    ret
```
<br></br>


### A test for Pie Chart Views:
```swift
func testPieChartView() {
        // Given:
        let frame = CGRect(x: 0, y: 0, width: 200.0, height: 200.0)
        let chartColors: [kSegmentColor] = [.protein, .carbs, .fat]
        let segments: [FBSegment] = [
            FBSegment(color: chartColors[0].color, value: 10.0),
            FBSegment(color: chartColors[1].color, value: 60.0),
            FBSegment(color: chartColors[2].color, value: 30.0),
        ]
        chartView       = FBPieGraphUIView(frame: frame, arrayOfSegments: segments)
//        for chartColor in chartColors {
//            (chartView as! GraphType).segments.append(FBSegment(color: chartColor.color, value: 0.0))
//        }
        
        (chartView as! GraphType).segments = segments
        
        // When:
        let chartValuesTotal = (chartView as! GraphType).segments.map({$0.value}).reduce(0){$0 + $1}
        
        let segmentsValuesTotal = segments.map({$0.value}).reduce(0){$0 + $1}
        
        
        // Then:
        XCTAssert(chartValuesTotal == segmentsValuesTotal, "Total of chartView values not same as 'segments' value total.")
    }
```
<br></br>


### A test to confirm that chart bar heights are not negative:
```swift
func testDrawGraphDataSegments_barHeightIsNotNegative() {
        // Given
        let barWidth        = 30.0
        let spacing         = 10.0
        let offset          = 30.0
        let yLabelHeight    = 20.0
        let viewHeight      = 400.0
        let xLabelHeight    = yLabelHeight
        
        let lineWidth       = 2.0
        var barHeight       = 0.0                              // initialize with zero value
        var y               = lineWidth - 1.0 + xLabelHeight   // start half way up width of axis line
        
        let brownColor      = UIColor.brown
        let segments        = [ FBSegment(color: brownColor, value: -1),
                                FBSegment(color: brownColor, value: 0),
                                FBSegment(color: brownColor, value: 10),
                                FBSegment(color: brownColor, value: 100),
                                FBSegment(color: brownColor, value: 1000_000_000)
        ]
        
        guard let maxValue  = segments.map({Double($0.value)}).max() else { return }
        let maxBarHeight    = maxValue > 0 ? maxValue : 100.0   // protects against dividing by zero.
        var barValues: [CGFloat] = []
        
        // When
        // Loop through the values array:
        for (index, segment) in segments.enumerated() {
            let i: CGFloat = CGFloat(index)
            let x: CGFloat = i * (barWidth + spacing) + offset + yLabelHeight
            
            let value   = segment.value > 0 ? segment.value : 0
            
            barHeight   = (value/maxBarHeight * (viewHeight - xLabelHeight))
            _           = CGRect(x: x, y: y, width: barWidth, height: barHeight)
            barValues.append(barHeight)
            
            y = lineWidth - 1.0 + xLabelHeight     // reset y
        }
        
        guard let minBarValue = barValues.min() else { return }
        
        
        // Then
        XCTAssert(minBarValue >= 0, "'barHeight' is not greater than or equal to zero.")
    }
```
<br></br>


### Testing FatSecretClient network API calls, when searching recipe IDs:

```swift
/// Test FatSecretClient network call for searching for recipe ID.
    func testFatSecretClient_searchRecipeID() {
        // Given
        let fatSecretClient = FatSecretClient()
        var myFspSingleRecipe: FSPSingleRecipe?
        let asyncSingleRecipeSearchExpectation = expectation(description: "asyncSingleRecipeSearchExpectation")
        
        // When
        // NB: 'certPinningDelegate:' is an optional argument with default nil value for .getRecipe.
        fatSecretClient.getRecipe(id: "187769") { recipe in
            switch recipe {
            case .success(let fspSingleRecipe):
//                let recipeInfo = fspSingleRecipe.recipe_description
//                let recipeID = fspSingleRecipe.recipe_id
//                print("food id info: \(recipeInfo)\n")
//                print("\nGet Recipe ID 187769...")
                myFspSingleRecipe = fspSingleRecipe
                
            case .failure(let failure):
                print("Single Recipe Search Failed with error: \(failure)")
            }
            
            asyncSingleRecipeSearchExpectation.fulfill()
        }
        
        // Then
        self.wait(for: [asyncSingleRecipeSearchExpectation], timeout: 10)
        XCTAssert(myFspSingleRecipe?.recipe_id == "187769", "FSP Food searched ID was not 187769.")
    }
```
<br></br>

# Sample Screen Shots
<table>
<tr>
	<td>
	<img src="images/screenshots/01-1a Search Main Screen Foods Trial Info.png" alt="iFoodSearch search screen" width="180"/>
	</td>
	<td>
	<img src="images/screenshots/01-1c Search Main Screen Foods Flipped BackView.png" alt="iFoodSearch search screen showing flipped image" width="180"/>
	</td>
	<td>
	<img src="images/screenshots/03-6a Food Info Potatoes ABS.png" alt="iFoodSearch Potato Details" width="180"/>
	</td>
	<td>
	<img src="images/screenshots/03-6b Food Info Potatoes ABS Dark Mode.png" alt="iFoodSearch Potato Details in Dark Mode" width="180"/>
	</td>
</tr>
<tr>
	<td>
	<img src="images/screenshots/06 Settings Pick Top Watched.png" alt="iFoodSearch Settings Top Watched" width="180"/>
	</td>
	<td>
	<img src="images/screenshots/07 Settings DRV for Nutrients.png" alt="iFoodSearch Settings Top DRV Values" width="180"/>
	</td>
	<td>
	<img src="images/screenshots/08 Recipe Details Pumpkin Pie.png" alt="iFoodSearch Recipe Details" width="180"/>
	</td>
	<td>
	<img src="images/screenshots/13 iFoodSearch Settings Bottom Part.png" alt="iFoodSearch Settings Bottom Part" width="180"/>
	</td>
</tr>
</table>
<br></br>

## This App required a lot of passion to keep it going... I've implemented the following:
#### Code Structure
* Model-View-Controller Design Pattern
* Delegates and Protocols
* Core Data Models using Generics
* Custom-built Charts, using Core Graphics
<br></br>

#### Testing/Error Handling
* Unit Testing
* Empty States
* Random Generation of Sample Test Data
* Text Input Validation
<br></br>

#### Security
* Image Asset Validation, in Assembly
* Integrated SecuritySuite and OpenSSL APIs
* UserDefaults in Keychain
* Certificate Pinning implementation
* StoreKit Receipt Validation
* Username/Password validation
<br></br>

#### Customization
* Light and Dark Mode Selections
* Theme Color Selections
* Custom Threshold Values
* Screen Startup Options
* Text Highlighting Options
<br></br>

#### Project Organization
* Code Documentation (DocC)
* Project group folder organization
* Privacy Manifest
* On-Demand Resources
<br></br>

# Future Considerations
### And yet, there's always more...
* Implement Core Data's CloudKit syncing, to sync with my macOS App iFoodTrack
* InterOp with SwiftUI, to implement SwiftCharts for newer iOS versions
* InterOp with SwiftUI, to implement Apple's StoreKitView API, for newer iOS versions
<br></br>


# FeedBack
<p class="contact-message">If you have any feedback or suggestions you can reach out via <a class="btn" href="mailto:fbotlogic@fbotlogicsolutions.com?subject=Blue Marble Weather Support">Email</a>.</p>
