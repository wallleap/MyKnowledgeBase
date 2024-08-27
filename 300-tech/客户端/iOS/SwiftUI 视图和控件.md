---
title: SwiftUI 视图和控件
date: 2024-08-21T12:09:22+08:00
updated: 2024-08-21T12:10:09+08:00
dg-publish: false
---

## Text[​](https://goswiftui.com/swiftui-views-and-controls/#text "Text的直接链接")

显示一行或多行只读文本的视图。

```
Text("Hello World")
```

样式

```
Text("Hello World")  .bold()  .italic()  .underline()  .lineLimit(2)
```

`Text` 中提供的字符串也用作 `LocalizedStringKey`，所以您可以自由获得 `NSLocalizedString` 的行为。

```
Text("This text used as localized key")
```

在文本视图中格式化文本。 实际上这不是 SwiftUI 功能，而是 Swift 5 字符串插值。

```
static let dateFormatter: DateFormatter = {    let formatter = DateFormatter()    formatter.dateStyle = .long    return formatter}()var now = Date()var body: some View {    Text("What time is it?: \(now, formatter: Self.dateFormatter)")}
```

您还可以将 `Text` 与 `+` 连接在一起。

```
Text("Hello ") + Text("World!").bold()
```

文本对齐。

```
Text("Hello\nWorld!").multilineTextAlignment(.center)
```

[文档 - Text](https://developer.apple.com/documentation/swiftui/text)

## Label[​](https://goswiftui.com/swiftui-views-and-controls/#label "Label的直接链接")

<div style="display: inline-block; background: #EB4C64; color: #fff; border-radius: 8px; padding: 2px 8px; font-size: 14px;">iOS 14</div>

标签是一种方便的视图，可以将图像和文本并排显示。 这个适用于菜单项或设置。

您可以使用自己的图像或 SF Symbol。

```
Label("Swift", image: "swift")Label("Website", systemImage: "globe")
```

我找不到调整图像大小的方法，并且 SF Symbols 与文本未对齐 在第一个测试版中。

[文档 - Label](https://developer.apple.com/documentation/swiftui/label)

## TextField[​](https://goswiftui.com/swiftui-views-and-controls/#textfield "TextField的直接链接")

显示可编辑文本界面的控件。

```
@State var name: String = "John"var body: some View {    TextField("Name's placeholder", text: $name)        .textFieldStyle(RoundedBorderTextFieldStyle())        .padding()}
```

[文档 - TextField](https://developer.apple.com/documentation/swiftui/textfield)

## TextEditor[​](https://goswiftui.com/swiftui-views-and-controls/#texteditor "TextEditor的直接链接")

iOS 14

一个可以显示和编辑长文本的视图。

```
@State private var fullText: String = "This is some editable text..."var body: some View {    TextEditor(text: $fullText)}
```

[文档 - TextEditor](https://developer.apple.com/documentation/swiftui/texteditor)

## SecureField[​](https://goswiftui.com/swiftui-views-and-controls/#securefield "SecureField的直接链接")

用户可以安全地输入密码或者私有文本的控件。

```
@State var password: String = "1234"var body: some View {    SecureField($password)        .textFieldStyle(RoundedBorderTextFieldStyle())        .padding()}
```

[文档 - SecureField](https://developer.apple.com/documentation/swiftui/securefield)

## Image[​](https://goswiftui.com/swiftui-views-and-controls/#image "Image的直接链接")

显示环境相关图像的视图。

```
Image("foo") //image name is foo
```

我们可以使用新的 SF 符号

```
Image(systemName: "clock.fill")
```

您可以为系统图标集添加样式以匹配您使用的字体

```
Image(systemName: "cloud.heavyrain.fill") .foregroundColor(.red) .font(.title)Image(systemName: "clock") .foregroundColor(.red) .font(Font.system(.largeTitle).bold())
```

给图片添加样式

```
Image("foo") .resizable() // it will sized so that it fills all the available space .aspectRatio(contentMode: .fit)
```

[文档 - Image](https://developer.apple.com/documentation/swiftui/image)

## Button[​](https://goswiftui.com/swiftui-views-and-controls/#button "Button的直接链接")

触发时执行操作的控件。

```
Button( action: { // did tap }, label: { Text("Click Me") })
```

如果你的 `Button` 的标签只是 `Text` 你可以用初始化 这个更简单的声明。

```
Button("Click Me") { // did tap}
```

你可以用这个按钮变得有点漂亮

```
Button(action: {}, label: { Image(systemName: "clock") Text("Click Me") Text("Subtitle")}).foregroundColor(Color.white).padding().background(Color.blue).cornerRadius(5)
```

[文档 - Button](https://developer.apple.com/documentation/swiftui/button)

## Link[​](https://goswiftui.com/swiftui-views-and-controls/#link "Link的直接链接")

iOS 14

创建一个链接样式的 [按钮](https://goswiftui.com/#button)，它将在关联的应用程序（如果可能），否则在用户的默认网络浏览器打开。

```
Link("View Our destination: URL(string: "https://www.example.com/TOS.html")!)
```

[文档 - Link](https://developer.apple.com/documentation/swiftui/link)

## NavigationLink[​](https://goswiftui.com/swiftui-views-and-controls/#navigationlink "NavigationLink的直接链接")

按下时触发导航演示的按钮。 可替换 `pushViewController`

```
NavigationView                                            { NavigationLink(destination: Text("Detail") .navigationBarTitle(Text("Detail")) ) { Text("Push") }.navigationBarTitle(Text("Master"))}
```

或者通过在自己的视图 `DetailView` 中使用组目标来使其更具可读性

```
NavigationView {    NavigationLink(destination: DetailView()) {        Text("Push")    }.navigationBarTitle(Text("Master"))}
```

不确定这是错误还是如此设计，在 Beta 5 中，上述代码将无法工作。 尝试换行像这样在 `List` 中的 `NavigationLink` 来测试此功能。

```
NavigationView {    List {        NavigationLink(destination: Text("Detail")) {            Text("Push")        }.navigationBarTitle(Text("Master"))    }}
```

如果你的 `NavigationLink` 的标签只有 `Text` 你可以 使用这个更简单的声明进行初始化。

```
NavigationLink("Detail", destination: Text("Detail").navigationBarTitle(Text("Detail")))
```

[文档 - NavigationLink](https://developer.apple.com/documentation/swiftui/navigationlink)

## ToolbarItem[​](https://goswiftui.com/swiftui-views-and-controls/#toolbaritem "ToolbarItem的直接链接")

iOS 14

表示可以放置在工具栏或导航栏中的项目的模型。 这代表了 `UINavigationItem`

中的大多数属性

添加 `titleView`。

```
NavigationView {    Text("SwiftUI").padding()        .toolbar {            ToolbarItem(placement: .principal) {                VStack {                    Text("Title")                    Button("Clickable Subtitle") { print("principle") }                }            }        }}
```

添加 `LeftBarButtonItem` 或 `LeftBarButtonItems`

```
NavigationView {    Text("SwiftUI").padding()        .toolbar {            ToolbarItem(placement: .navigationBarLeading) {                Button {                } label: {                    Image(systemName: "square.and.pencil")                }            }        }}
```

添加 `rightBarButtonItem` 或 `rightBarButtonItems`。

```
NavigationView {    Text("SwiftUI").padding()        .toolbar {            ToolbarItem(placement: .primaryAction) {                Button {                } label: {                    Image(systemName: "square.and.pencil")                }            }            ToolbarItem(placement: .navigationBarTrailing) {                Button {                } label: {                    Image(systemName: "square.and.pencil")                }            }        }}
```

[文档 - ToolbarItem](https://developer.apple.com/documentation/swiftui/toolbaritem)

## Toggle[​](https://goswiftui.com/swiftui-views-and-controls/#toggle "Toggle的直接链接")

在开启和关闭状态之间切换的控件。

```
@State var isShowing = true // toggle stateToggle(isOn: $isShowing) {    Text("Hello World")}
```

如果你的 `Toggle` 的标签只是 `Text` 你可以用 这个更简单的声明。

```
Toggle("Hello World", isOn: $isShowing)
```

[文档 - Toggle](https://developer.apple.com/documentation/swiftui/toggle)

## Map[​](https://goswiftui.com/swiftui-views-and-controls/#map "Map的直接链接")

iOS 14

显示嵌入式地图界面的视图。

显示指定区域的地图

```
import MapKit@State var region = MKCoordinateRegion(center: .init(latitude: 37.334722, longitude: -122.008889), latitudinalMeters: 300, longitudinalMeters: 300)Map(coordinateRegion: $region)
```

您可以通过指定 `interactionModes` 来控制地图交互（使用 `[]` 禁用所有交互）。

```
struct PinItem: Identifiable {    let id = UUID()    let coordinate: CLLocationCoordinate2D}Map(coordinateRegion: $region,    interactionModes: [],    showsUserLocation: true,    userTrackingMode: nil,    annotationItems: [PinItem(coordinate: .init(latitude: 37.334722, longitude: -122.008889))]) { item in    MapMarker(coordinate: item.coordinate)}
```

[文档 - Map](https://developer.apple.com/documentation/mapkit/map)

## Picker[​](https://goswiftui.com/swiftui-views-and-controls/#picker "Picker的直接链接")

用于从一组互斥值中进行选择的控件。

选择器样式根据其祖先进行更改，在 `Form` 或 `List` 下，它显示为单个列表行，您可以点击它以进入一个显示所有可能选项的新屏幕。

```
NavigationView {    Form {        Section {            Picker(selection: $selection, label:                Text("Picker Name")                , content: {                    Text("Value 1").tag(0)                    Text("Value 2").tag(1)                    Text("Value 3").tag(2)                    Text("Value 4").tag(3)            })        }    }}
```

您可以使用 `.pickerStyle(WheelPickerStyle())` 覆盖样式。

在 SwiftUI 中，`UISegmentedControl` 是另一种风格 `选择器`。

```
@State var mapChoioce = 0var settings = ["Map", "Transit", "Satellite"]Picker("Options", selection: $mapChoioce) {    ForEach(0 ..< settings.count) { index in        Text(self.settings[index])            .tag(index)    }}.pickerStyle(SegmentedPickerStyle())
```

分段控制在 iOS 13 中也焕然一新

[文档 - Picker](https://developer.apple.com/documentation/swiftui/picker)

## DatePicker[​](https://goswiftui.com/swiftui-views-and-controls/#datepicker "DatePicker的直接链接")

用于选择日期的控件。

日期选择器样式也会根据其祖先而改变。 在 `Form` 下或 `List`，它显示为单个列表行，您可以点击以展开日期选择器（就像日历应用程序一样）。

```
@State var selectedDate = Date()var dateClosedRange: ClosedRange<Date> {    let min = Calendar.current.date(byAdding: .day, value: -1, to: Date())!    let max = Calendar.current.date(byAdding: .day, value: 1, to: Date())!    return min...max}NavigationView {    Form {        Section {            DatePicker(                selection: $selectedDate,                in: dateClosedRange,                displayedComponents: .date,                label: { Text("Due Date") }            )        }    }}
```

在 `Form` 和 `List` 之外，它显示为普通的滑动选择器

```
@State var selectedDate = Date()var dateClosedRange: ClosedRange<Date> {    let min = Calendar.current.date(byAdding: .day, value: -1, to: Date())!    let max = Calendar.current.date(byAdding: .day, value: 1, to: Date())!    return min...max}DatePicker(    selection: $selectedDate,    in: dateClosedRange,    displayedComponents: [.hourAndMinute, .date],    label: { Text("Due Date") })
```

如果你的 `DatePicker` 的标签只是纯 `Text` 你可以使用这个更简单的声明进行初始化。

```
DatePicker("Due Date",            selection: $selectedDate,            in: dateClosedRange,            displayedComponents: [.hourAndMinute, .date])
```

`minimumDate` 和 `maximumDate` 可以使用 [`ClosedRange`](https://developer.apple.com/documentation/swift/closedrange), [`PartialRangeThrough`](https://developer.apple.com/documentation/swift/partialrangethrough), 和 [`PartialRangeFrom`](https://developer.apple.com/documentation/swift/partialrangefrom).

```
DatePicker("Minimum Date",    selection: $selectedDate,    in: Date()...,    displayedComponents: [.date])DatePicker("Maximum Date",    selection: $selectedDate,    in: ...Date(),    displayedComponents: [.date])
```

[文档 - DatePicker](https://developer.apple.com/documentation/swiftui/datepicker)

## ProgressView[​](https://goswiftui.com/swiftui-views-and-controls/#progressview "ProgressView的直接链接")

iOS 14

显示任务完成进度的视图。

```
@State private var progress = 0.5VStack { ProgressView(value: progress) Button("More", action: { progress += 0.05 })}
```

通过应用 `CircularProgressViewStyle` 可以将其用作 `UIActivityIndicatorView`。

```
ProgressView(value: progress) .progressViewStyle(CircularProgressViewStyle())
```

[文档 - ProgressView](https://developer.apple.com/documentation/swiftui/progressview)

## Slider[​](https://goswiftui.com/swiftui-views-and-controls/#slider "Slider的直接链接")

在取值范围内滑动选择值的控件。

```
@State var progress: Float = 0Slider(value: $progress, from: 0.0, through: 100.0, by: 5.0)
```

滑块缺少 `minimumValueImage` 和 `maximumValueImage`，但是 我们可以通过 `HStack

轻松复制

```
@State var progress: Float = 0HStack {    Image(systemName: "sun.min")    Slider(value: $progress, from: 0.0, through: 100.0, by: 5.0)    Image(systemName: "sun.max.fill")}.padding()
```

[文档 - Slider](https://developer.apple.com/documentation/swiftui/slider)

## Stepper[​](https://goswiftui.com/swiftui-views-and-controls/#stepper "Stepper的直接链接")

用于执行语义递增和递减动作的控件。

```
@State var quantity: Int = 0Stepper(value: $quantity, in: 0...10, label: { Text("Quantity \(quantity)")})
```

如果你的 `Stepper` 的标签只有 `Text` 你可以初始化使用这个更简单的声明。

```
Stepper("Quantity \(quantity)", value: $quantity, in: 0...10)
```

如果您想要完全控制，他们会提供简单的 `Stepper` 供您管理您自己的数据源。

```
@State var quantity: Int = 0Stepper(onIncrement: {    self.quantity += 1}, onDecrement: {    self.quantity -= 1}, label: { Text("Quantity \(quantity)") })
```

如果您还使用初始化器为每个步骤指定一个值的数量 `step`。

```
Stepper(value: $quantity, in: 0...10, step: 2) {    Text("Quantity \(quantity)")}
```

[文档 - Stepper](https://developer.apple.com/documentation/swiftui/stepper)
