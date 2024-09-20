# Code39 Barcode Generator for Power BI Report Builder

This project provides a solution for generating Code 39 barcodes within Power BI Report Builder. It uses VB.NET code to generate the barcode pattern and visualizes it using a series of colored rectangles.

## How it Works

1. **Input Processing**: The input is trimmed to 6 digits, removing any non-numeric characters.
2. **Code 39 Generation**: Each digit is converted to its corresponding Code 39 pattern.
3. **Barcode Visualization**: The barcode is represented by a series of 124 rectangles, colored black or white based on the generated pattern.

## Barcode Structure

For a 6-digit input, the barcode structure is as follows:

- Start quiet zone: 10 spaces
- Start character (*): 13 bars/spaces
- 6 digits: 6 * 13 = 78 bars/spaces
- Stop character (*): 13 bars/spaces
- End quiet zone: 10 spaces

Total: 10 + 13 + 78 + 13 + 10 = 124 bars/spaces

## Implementation

### Code

Add the following VB.NET code to your report:

```vb.net
Function ProcessInput(ByVal data As String) As String
    data = Trim(data)
    data = System.Text.RegularExpressions.Regex.Replace(data, "[^\d]", "")
    If Len(data) > 6 Then
        data = Right(data, 6)  ' Take the last 6 digits
    End If
    Return data
End Function

Function Code39(ByVal data As String) As String
    Dim patterns As New System.Collections.Generic.Dictionary(Of Char, String)()
    patterns.Add("0"c, "101001101101")
    patterns.Add("1"c, "110100101011")
    patterns.Add("2"c, "101100101011")
    patterns.Add("3"c, "110110010101")
    patterns.Add("4"c, "101001101011")
    patterns.Add("5"c, "110100110101")
    patterns.Add("6"c, "101100110101")
    patterns.Add("7"c, "101001011011")
    patterns.Add("8"c, "110100101101")
    patterns.Add("9"c, "101100101101")
    patterns.Add("*"c, "100101101101")
    Dim result As String = patterns("*"c)
    For Each c As Char In data
        If patterns.ContainsKey(c) Then
            result &= "0" & patterns(c) ' Add a space between characters
        End If
    Next
    result &= patterns("*"c)
    Return result
End Function

Function GenerateBarcode(ByVal data As String) As String
    data = ProcessInput(data)
    Dim barcode As String = Code39(data)
    Dim result As String = New String("W"c, 10) ' Initial quiet zone
    For i As Integer = 1 To Len(barcode)
        result &= IIf(Mid(barcode, i, 1) = "1", "B", "W")
    Next
    result &= New String("W"c, 10) ' Final quiet zone
    Return result
End Function

Function GetBarcodeColor(ByVal data As String, ByVal index As Integer) As String
    Dim barcode As String = GenerateBarcode(data)
    If index >= 1 AndAlso index <= Len(barcode) Then
        Return IIf(Mid(barcode, index, 1) = "B", "Black", "White")
    Else
        Return "White"
    End If
End Function

Function GetDebugInfo(ByVal data As String) As String
    Dim processedData = ProcessInput(data)
    Dim code39Pattern = Code39(processedData)
    Dim barcodePattern = GenerateBarcode(processedData)
    Dim debugInfo As String = ""
    debugInfo &= "Raw Input: " & data & vbNewLine
    debugInfo &= "Processed Input: " & processedData & vbNewLine
    debugInfo &= "Code 39 Pattern: " & code39Pattern & vbNewLine
    debugInfo &= "Code 39 Pattern Length: " & Len(code39Pattern).ToString() & vbNewLine
    debugInfo &= "Final Barcode Pattern: " & barcodePattern & vbNewLine
    debugInfo &= "Final Pattern Length: " & Len(barcodePattern).ToString() & vbNewLine
    debugInfo &= "Character Breakdown:" & vbNewLine
    For i As Integer = 1 To Len(processedData)
        Dim charPattern = Mid(code39Pattern, (i-1)*13 + 13, 13)
        debugInfo &= "  " & Mid(processedData, i, 1) & ": " & charPattern & vbNewLine
    Next
    Return debugInfo
End Function
```

### Report Design

1. Create 124 equal-sized rectangles and place them side by side.
2. For each rectangle, add the following expression in the BackgroundColor property, where X is the rectangle number (1 to 124):
   ```
   =Code.GetBarcodeColor(Fields!YourNumericField.Value.ToString(), X)
   ```
3. Add a text field to display the debug information:
   ```
   =Code.GetDebugInfo(Fields!YourNumericField.Value.ToString())
   ```

## Usage

1. In your Power BI Report Builder, create a new report or open an existing one.
2. Add the VB.NET code provided above to your report's Code section.
3. Design your report layout as described in the Report Design section.
4. Run the report, providing a numeric input field for the barcode.

The barcode will be generated based on the input, and debug information will be displayed to help understand the barcode structure.

## Notes

- This implementation is designed for 6-digit numeric inputs. Adjust the code if you need to support different input lengths or character types.
- Ensure that the rectangles are thin enough to create a readable barcode when printed or displayed.
- The debug information is useful for understanding how the barcode is generated but can be removed in production reports.

## Contributing

Feel free to fork this project and submit pull requests with improvements or bug fixes. For major changes, please open an issue first to discuss what you would like to change.

