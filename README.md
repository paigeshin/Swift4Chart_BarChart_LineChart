# Swift4Chart_BarChart_LineChart

```swift

import SwiftUI
import Charts

struct SiteView: Identifiable {
    let id = UUID().uuidString
    var hour: Date
    var views: Double
    var animate: Bool = false
}

extension Date {
    
    func updatedHour(value: Int) -> Date {
        return Calendar.current.date(bySettingHour: value, minute: 0, second: 0, of: self) ?? .now
    }
    
}

var sample_analaytics: [SiteView] = [
    SiteView(hour: Date().updatedHour(value: 8), views: Double(Int.random(in: 1500...9000))),
    SiteView(hour: Date().updatedHour(value: 9), views: Double(Int.random(in: 1500...9000))),
    SiteView(hour: Date().updatedHour(value: 10), views: Double(Int.random(in: 1500...9000))),
    SiteView(hour: Date().updatedHour(value: 11), views: Double(Int.random(in: 1500...9000))),
    SiteView(hour: Date().updatedHour(value: 12), views: Double(Int.random(in: 1500...9000))),
    SiteView(hour: Date().updatedHour(value: 13), views: Double(Int.random(in: 1500...9000))),
    SiteView(hour: Date().updatedHour(value: 14), views: Double(Int.random(in: 1500...9000))),
    SiteView(hour: Date().updatedHour(value: 15), views: Double(Int.random(in: 1500...9000))),
    SiteView(hour: Date().updatedHour(value: 16), views: Double(Int.random(in: 1500...9000))),
    SiteView(hour: Date().updatedHour(value: 17), views: Double(Int.random(in: 1500...9000))),
    SiteView(hour: Date().updatedHour(value: 18), views: Double(Int.random(in: 1500...9000))),
    SiteView(hour: Date().updatedHour(value: 19), views: Double(Int.random(in: 1500...9000))),
    SiteView(hour: Date().updatedHour(value: 20), views: Double(Int.random(in: 1500...9000))),
    SiteView(hour: Date().updatedHour(value: 21), views: Double(Int.random(in: 1500...9000))),
]


struct ContentView: View {
    
    @State var sampleAnalytics: [SiteView] = sample_analaytics
    
    @State var currentTab: String = "7 Days"
    
    // For Rule Mark
    @State var currentActiveItem: SiteView?
    // Rule Mark to make it position at the center of the bar => plotWidth / 2
    @State var plotWidth: CGFloat = 0

    
    var body: some View {
        NavigationStack {
            VStack {
                // MARK: NEW CHART API
                VStack(alignment: .leading, spacing: 12) {
                    
                    HStack {
                        
                    
                        Text("Views")
                            .fontWeight(.semibold)
                        
                        
                        Picker("", selection: self.$currentTab) {
                            Text("7 Days")
                                .tag("7 Days")
                            Text("Week")
                                .tag("Week")
                            Text("Month")
                                .tag("Month")
                        }
                        .pickerStyle(.segmented)
                        .padding(.leading, 80)
                    } //: HSTACK
                    
                    let totalValue = self.sampleAnalytics.reduce(0.0) { partialResult, item in
                        item.views + partialResult
                    }
                    
                    Text(totalValue.stringFormat)
                        .font(.largeTitle.bold())
                    
                    LineChart()
                } //: VSTACK
                .padding()
                .background(
                    RoundedRectangle(cornerRadius: 10, style: .continuous)
                        .fill(.white.shadow(.drop(radius: 2)))
                )
         
                
            } //:VSTACK
            .frame(maxWidth: .infinity, maxHeight: .infinity, alignment: .top)
            .padding()
            .navigationTitle("Swift Charts")
            // MARK: Simply Updating Values For Segmented Tabs
            .onChange(of: self.currentTab) { tab in
                if tab != "7 Days" {
                    for (index, _) in self.sampleAnalytics.enumerated() {
                        self.sampleAnalytics[index].views = .random(in: 1500...10000)
                         
                    }
                }
                
                // Re-animating Values
                self.animateGraph()
            }
            
        } //: STACK
    }
    
    @ViewBuilder
    func BarChart() -> some View {
        let max: Double = self.sampleAnalytics.max(by: { $0.views > $1.views })?.views ?? 0
        Chart {
            ForEach(self.sampleAnalytics) { item in
                BarMark(x: .value("Hour", item.hour, unit: .hour),
                        y: .value("Views", item.animate ? item.views : 0 ))
                .foregroundStyle(.blue.gradient)
                    
                // MARK: Rule Mark For Currently Dragging Item
                if let currentActiveItem, currentActiveItem.id == item.id {
                    RuleMark(x: .value("Hour", currentActiveItem.hour))
                        .lineStyle(.init(lineWidth: 2, miterLimit: 2, dash: [2], dashPhase: 5))
                        // MARK: Setting In Middle Of Each Bars
                        .offset(x: (self.plotWidth / CGFloat(self.sampleAnalytics.count)) / 2)
                        .annotation(position: .top) {
                            VStack(alignment: .leading, spacing: 6) {
                                Text("Views")
                                    .font(.caption)
                                    .foregroundColor(.gray)
                                
                                Text(currentActiveItem.views.stringFormat)
                                    .font(.title3.bold())
                                
                            } //: VSTACK
                            .padding(.horizontal, 10)
                            .padding(.vertical, 4)
                            .background(
                                RoundedRectangle(cornerRadius: 6, style: .continuous)
                                    .fill(.white.shadow(.drop(radius: 2)))
                            )
                        }
                }
                
            }
            // Applying Gradient Style
            // From SwiftUI 4.0 we can direcly create gradient from color
        }
        //        .frame(height: 250)
        // MARK: Customizing Y-Axis Length
        .chartYScale(domain: 0...(max + 10000))
        // MARK: GESTURE TO HIGHLIGHT CURRENT BAR
        .chartOverlay(content: { proxy in
            GeometryReader { innerProxy in
                Rectangle()
                    .fill(.clear)
                    .contentShape(Rectangle())
                    .gesture(
                        DragGesture()
                            .onChanged({ value in
                                // MARK: GETTING CURRENT LOCATION
                                let location = value.location
                                // Extracting Value from the location
                                // Swift Charts gives the direct ability to do that
                                // We're going to extract the Date in A-Axis. Then with the help of that date value, we're extractiing the current item.
                                
                                // Don't forget to include the perfect data type
                                if let hour: Date = proxy.value(atX: location.x) {
                                    // Extracting hour
                                    let hour = Calendar.current.component(.hour, from: hour)
                                    print(hour)
                                    if let currentItem = self.sampleAnalytics.first(where: { item in
                                        Calendar.current.component(.hour, from: item.hour) == hour
                                    }) {
                                        self.currentActiveItem = currentItem
                                        self.plotWidth = proxy.plotAreaSize.width
                                    }
                                }
                                
                            })
                            .onEnded({ value in
                                self.currentActiveItem = nil
                            })
                    )
            }
        })
        .frame(height: 250)
        .onAppear {
            self.animateGraph()
        }
        
    }
        
    @ViewBuilder
    func LineChart() -> some View {
        let max: Double = self.sampleAnalytics.max(by: { $0.views > $1.views })?.views ?? 0
        Chart {
            ForEach(self.sampleAnalytics) { item in
                LineMark(x: .value("Hour", item.hour, unit: .hour),
                         y: .value("Views", item.animate ? item.views : 0 ))
                .foregroundStyle(.blue.gradient)
                .interpolationMethod(.catmullRom)
                
                AreaMark(x: .value("Hour", item.hour, unit: .hour),
                         y: .value("Views", item.animate ? item.views : 0 ))
                .foregroundStyle(.blue.opacity(0.1).gradient)
                .interpolationMethod(.catmullRom)
                    
                // MARK: Rule Mark For Currently Dragging Item
                if let currentActiveItem, currentActiveItem.id == item.id {
                    RuleMark(x: .value("Hour", currentActiveItem.hour))
                        .lineStyle(.init(lineWidth: 2, miterLimit: 2, dash: [2], dashPhase: 5))
                        // MARK: Setting In Middle Of Each Bars
                        .offset(x: (self.plotWidth / CGFloat(self.sampleAnalytics.count)) / 2)
                        .annotation(position: .top) {
                            VStack(alignment: .leading, spacing: 6) {
                                Text("Views")
                                    .font(.caption)
                                    .foregroundColor(.gray)
                                
                                Text(currentActiveItem.views.stringFormat)
                                    .font(.title3.bold())
                                
                            } //: VSTACK
                            .padding(.horizontal, 10)
                            .padding(.vertical, 4)
                            .background(
                                RoundedRectangle(cornerRadius: 6, style: .continuous)
                                    .fill(.white.shadow(.drop(radius: 2)))
                            )
                        }
                }
                
            }
            // Applying Gradient Style
            // From SwiftUI 4.0 we can direcly create gradient from color
        }
        //        .frame(height: 250)
        // MARK: Customizing Y-Axis Length
        .chartYScale(domain: 0...(max + 10000))
        // MARK: GESTURE TO HIGHLIGHT CURRENT BAR
        .chartOverlay(content: { proxy in
            GeometryReader { innerProxy in
                Rectangle()
                    .fill(.clear)
                    .contentShape(Rectangle())
                    .gesture(
                        DragGesture()
                            .onChanged({ value in
                                // MARK: GETTING CURRENT LOCATION
                                let location = value.location
                                // Extracting Value from the location
                                // Swift Charts gives the direct ability to do that
                                // We're going to extract the Date in A-Axis. Then with the help of that date value, we're extractiing the current item.
                                
                                // Don't forget to include the perfect data type
                                if let hour: Date = proxy.value(atX: location.x) {
                                    // Extracting hour
                                    let hour = Calendar.current.component(.hour, from: hour)
                                    print(hour)
                                    if let currentItem = self.sampleAnalytics.first(where: { item in
                                        Calendar.current.component(.hour, from: item.hour) == hour
                                    }) {
                                        self.currentActiveItem = currentItem
                                        self.plotWidth = proxy.plotAreaSize.width
                                    }
                                }
                                
                            })
                            .onEnded({ value in
                                self.currentActiveItem = nil
                            })
                    )
            }
        })
        .frame(height: 250)
        .onAppear {
            self.animateGraph()
        }
        
    }
    
    // MARK: ANIMATING GRAPH
    func animateGraph() {
        for (index, _ ) in self.sampleAnalytics.enumerated() {
            
            // For Some Reason Delay is Not Working
            // Using Dispatch Queue Delay
            DispatchQueue.main.asyncAfter(deadline: .now() + Double(index) * 0.05) {
                withAnimation(.interactiveSpring(response: 0.8, dampingFraction: 0.8, blendDuration: 0.8)) {
                    self.sampleAnalytics[index].animate = true
                }
            }
        }
    }
    
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}

// MARK: EXTENSION TO CONVERT DOUBLE TO STRING WITH K,M NUMBER VALUES
// EG: 10K, 10M, ... ETC
extension Double {
    
    var stringFormat: String {
        if self > 10000 && self < 999999 {
            return String(format: "%.1fK", self / 10000).replacingOccurrences(of: ".0", with: "")
        }
        if self > 999999 {
            return String(format: "%.1fM", self / 1000000).replacingOccurrences(of: ".0", with: "")
        }
        return String(format: "%.0f", self)
    }
    
}


```
