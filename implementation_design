# Floor Plan Import & AI Recognition System Design

## ภาพรวมระบบ (System Overview)

ระบบนี้มีเป้าหมายเพื่อ **นำเข้ารูปภาพแผนผังสถานที่ (Floor Plan)** และใช้ AI วิเคราะห์เพื่อ **แปลงเป็นข้อมูลดิจิทัล** ที่สามารถนำไปใช้แสดงผลแบบ Interactive ได้

![ตัวอย่างแผนผังที่นำเข้า](C:/Users/USER/.gemini/antigravity/brain/f15cba1e-0ad2-4bbc-8f14-010e28b489c4/uploaded_image_1768503516334.png)

---

## 🎯 เป้าหมายหลัก (Objectives)

1. **Object Detection** - ตรวจจับโต๊ะ/ที่นั่งทุกตัวในแผนผัง
2. **Text Recognition (OCR)** - อ่านชื่อโต๊ะ/โซน (เช่น A1, B3, VIP, STAGE)
3. **Position Mapping** - หาตำแหน่ง (x, y) และขนาดของแต่ละ object
4. **Zone Classification** - แยกประเภทโซน (STAGE, BAR, VIP, ห้องน้ำ ฯลฯ)
5. **Export JSON** - ส่งออกข้อมูลในรูปแบบที่แอปใช้งานได้

---

## 🛠️ Technology Stack Comparison

### Option 1: OpenAI GPT-4 Vision (⭐ แนะนำสำหรับ Prototype)

```mermaid
flowchart LR
    A[Floor Plan Image] --> B[OpenAI GPT-4o API]
    B --> C[Structured JSON Output]
    C --> D[Flutter App]
```

| ข้อดี | ข้อเสีย |
|-------|---------|
| เข้าใจ Context ได้ดี (รู้ว่า STAGE คืออะไร) | ราคาสูงกว่า (~$0.01-0.03/image) |
| รองรับภาษาไทยบน Image | ความแม่นยำ coordinates อาจคลาดเคลื่อน |
| ไม่ต้อง Train Model | ต้อง post-process ผลลัพธ์ |
| สามารถ request JSON format ได้โดยตรง | Rate limit ต่ำกว่า custom model |

**API Request Example:**
```javascript
const response = await openai.chat.completions.create({
  model: "gpt-4o",
  messages: [{
    role: "user",
    content: [
      { type: "text", text: `Analyze this floor plan image and extract all tables/seats. 
        Return JSON with format: {
          "zones": [{ "id": "STAGE", "type": "stage", "bbox": [x,y,w,h] }],
          "tables": [{ "id": "A1", "zone": "A", "bbox": [x,y,w,h], "type": "table" }]
        }` 
      },
      { type: "image_url", image_url: { url: base64Image } }
    ]
  }],
  response_format: { type: "json_object" }
});
```

---

### Option 2: Google Cloud Vision + Custom Processing (⭐⭐ แนะนำสำหรับ Production)

```mermaid
flowchart LR
    A[Floor Plan Image] --> B[Cloud Vision OCR]
    A --> C[Object Detection API]
    B --> D[Text Processor]
    C --> D
    D --> E[Position Calculator]
    E --> F[Structured JSON]
```

| Components | Purpose |
|------------|---------|
| **Cloud Vision OCR** | ดึงข้อความทั้งหมดพร้อมตำแหน่ง |
| **Object Detection** | ใช้ AutoML Vision หรือ YOLOv8 ตรวจจับ rectangles |
| **Custom Processor** | จับคู่ text กับ bounding box |

**ราคา:**
- Cloud Vision OCR: $1.50/1000 images
- Object Detection: ขึ้นกับ model

---

### Option 3: RasterScan API (⭐⭐⭐ Specialized for Floor Plans)

> Production-ready API เฉพาะสำหรับ Floor Plan Recognition

```mermaid
flowchart LR
    A[Floor Plan Image] --> B[RasterScan API]
    B --> C[Walls, Doors, Symbols]
    C --> D[DXF/IFC/GLTF Export]
```

| Feature | Description |
|---------|-------------|
| **Wall Detection** | ตรวจจับผนัง ประตู หน้าต่าง |
| **Symbol Recognition** | รู้จำ furniture symbols |
| **3D Model Export** | สร้าง 3D model จาก 2D plan |
| **Multiple Formats** | Export เป็น DXF, IFC, GLTF |

**เหมาะสำหรับ:** Floor plan ที่เป็นแบบบ้าน/อาคาร (architectural)

> [!WARNING]
> สำหรับ Event Venue Layout (แบบในตัวอย่าง) อาจไม่เหมาะเท่า Option 1-2 เพราะไม่ได้ออกแบบมาสำหรับ tables/seats pattern

---

### Option 4: Custom YOLOv8 + PaddleOCR (⭐⭐⭐ Highest Accuracy)

```mermaid
flowchart TD
    A[Training Data] --> B[Label Tables/Zones]
    B --> C[Train YOLOv8]
    D[Floor Plan Image] --> E[YOLOv8 Detection]
    D --> F[PaddleOCR]
    E --> G[Merge Results]
    F --> G
    G --> H[JSON Output]
```

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Object Detection** | YOLOv8 / YOLO11 | ตรวจจับ bounding boxes |
| **Text Recognition** | PaddleOCR / EasyOCR | อ่านข้อความภาษาไทย+อังกฤษ |
| **Inference Server** | Python + FastAPI | รัน model |
| **Model Hosting** | Replicate / RunPod | GPU inference |

**ข้อดี:**
- ความแม่นยำสูงสุดหลัง training
- ควบคุม classes ได้เอง (โต๊ะ, เวที, บาร์, ฯลฯ)
- ค่าใช้จ่ายต่ำในระยะยาว

**ข้อเสีย:**
- ต้อง label data เอง (ใช้ Roboflow หรือ Label Studio)
- ต้องมี GPU สำหรับ training
- ใช้เวลา setup นานกว่า

---

## 📊 Recommendation Matrix

| Criteria | GPT-4o | Cloud Vision | RasterScan | Custom YOLO |
|----------|--------|--------------|------------|-------------|
| **Setup Speed** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐ |
| **Accuracy** | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Thai Text Support** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Cost (per 1000 images)** | $10-30 | $1.50 | Varies | $0.50 (self-hosted) |
| **Customization** | ⭐⭐ | ⭐⭐⭐ | ⭐ | ⭐⭐⭐⭐⭐ |
| **Event Venue Suitability** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ |

---

## 🏆 Recommended Approach: Hybrid Solution

### Phase 1: Quick Start with GPT-4o

```typescript
// backend/src/routes/floor-plan.ts
import OpenAI from 'openai';

const analyzeFloorPlan = async (imageBase64: string) => {
  const response = await openai.chat.completions.create({
    model: "gpt-4o",
    messages: [{
      role: "system",
      content: `You are a floor plan analyzer. Extract all tables, seats, and zones from the image.
        Output JSON format:
        {
          "imageSize": { "width": number, "height": number },
          "zones": [{ 
            "id": string, 
            "label": string,
            "type": "stage"|"bar"|"vip"|"seating"|"facility"|"walkway",
            "color": string,
            "bbox": { "x": number, "y": number, "width": number, "height": number }
          }],
          "tables": [{
            "id": string,
            "zone": string,
            "type": "table"|"seat"|"standing",
            "bbox": { "x": number, "y": number, "width": number, "height": number }
          }]
        }`
    }, {
      role: "user", 
      content: [
        { type: "image_url", image_url: { url: `data:image/png;base64,${imageBase64}` } }
      ]
    }],
    response_format: { type: "json_object" }
  });
  
  return JSON.parse(response.choices[0].message.content);
};
```

### Phase 2: Fine-tune with Custom Detection

หลังจากรวบรวม Floor Plan images ได้มากพอ (50-100 images):

1. **Label Data** ด้วย Roboflow
2. **Train YOLOv8** สำหรับ:
   - `table` - โต๊ะทั่วไป
   - `vip_table` - โต๊ะ VIP
   - `stage` - เวที
   - `bar` - บาร์
   - `restroom` - ห้องน้ำ
3. **Deploy** บน Replicate หรือ Fly.io with GPU

### Phase 3: Full Pipeline

```mermaid
flowchart TD
    subgraph Frontend["Flutter App"]
        A[Upload Image] --> B[Preview]
        B --> C[Confirm Upload]
    end
    
    subgraph Backend["Bun/Hono API"]
        C --> D[Store Original Image]
        D --> E[Queue Analysis Job]
        E --> F{Analysis Engine}
    end
    
    subgraph AI["AI Processing"]
        F -->|Quick| G[GPT-4o Analysis]
        F -->|Accurate| H[YOLOv8 + PaddleOCR]
    end
    
    G --> I[Store Results]
    H --> I
    I --> J[Return JSON to App]
    J --> K[Render Interactive Map]
```

---

## 📐 Output JSON Schema

```typescript
interface FloorPlanAnalysisResult {
  id: string;
  originalImage: string; // URL to stored image
  imageSize: {
    width: number;
    height: number;
  };
  analyzedAt: string; // ISO timestamp
  
  zones: Zone[];
  tables: Table[];
  facilities: Facility[];
  metadata: {
    confidence: number;
    processingTimeMs: number;
    engine: 'gpt4o' | 'yolov8' | 'hybrid';
  };
}

interface Zone {
  id: string;
  label: string;
  labelThai?: string;
  type: 'stage' | 'bar' | 'vip' | 'seating' | 'standing' | 'walkway' | 'outdoor' | 'indoor';
  color: string; // hex color from image
  bbox: BoundingBox;
  tables?: string[]; // table IDs in this zone
}

interface Table {
  id: string; // e.g., "A1", "B12", "VIP-3"
  zone: string; // zone ID
  type: 'table' | 'sofa' | 'standing' | 'bar_seat';
  capacity?: number;
  bbox: BoundingBox;
  status?: 'available' | 'reserved' | 'sold';
}

interface Facility {
  id: string;
  type: 'restroom' | 'cashier' | 'entrance' | 'exit' | 'projector' | 'speaker' | 'pillar';
  label?: string;
  bbox: BoundingBox;
}

interface BoundingBox {
  x: number; // percentage 0-100
  y: number;
  width: number;
  height: number;
}
```

---

## 🔄 Coordinate System

เพื่อให้ตำแหน่งแม่นยำบนหน้าจอทุกขนาด ใช้ **Percentage-based Coordinates**:

```
Original Image: 1024 x 768 px
Table A1 position: (120, 200) px

→ Normalized: 
  x: 120 / 1024 = 0.117 (11.7%)
  y: 200 / 768 = 0.260 (26.0%)
```

**Flutter Rendering:**
```dart
Positioned(
  left: zone.bbox.x / 100 * containerWidth,
  top: zone.bbox.y / 100 * containerHeight,
  width: zone.bbox.width / 100 * containerWidth,
  height: zone.bbox.height / 100 * containerHeight,
  child: ZoneWidget(zone: zone),
)
```

---

## 💰 Cost Estimation

| Solution | Setup Cost | Per Image | 1000 Images/month |
|----------|-----------|-----------|-------------------|
| GPT-4o Only | $0 | $0.01-0.03 | $10-30 |
| Cloud Vision | $0 | $0.0015 | $1.50 |
| YOLOv8 (Replicate) | ~$50 training | $0.001 | $1 |
| YOLOv8 (Self-hosted) | ~$100 setup | ~$0.0005 | $0.50 |

---

## 🚀 Implementation Phases

### Phase 1: MVP (1-2 weeks)
- [ ] Integrate GPT-4o API endpoint
- [ ] Create Flutter upload flow
- [ ] Basic JSON parsing and display
- [ ] Manual position adjustment UI

### Phase 2: Accuracy Improvement (2-4 weeks)
- [ ] Collect 50+ floor plan samples
- [ ] Label data with Roboflow
- [ ] Train YOLOv8 custom model
- [ ] Add PaddleOCR for Thai text

### Phase 3: Production Polish (2-4 weeks)
- [ ] Caching & optimization
- [ ] Batch processing
- [ ] Admin dashboard for corrections
- [ ] A/B testing between engines

---

## 📝 สรุป (Summary)

สำหรับ **Event Venue Floor Plan** แบบในตัวอย่าง แนะนำ:

1. **เริ่มต้นด้วย GPT-4o** - Setup เร็ว, เข้าใจ context ดี
2. **เพิ่ม PaddleOCR** - สำหรับอ่านภาษาไทยที่แม่นยำกว่า
3. **Custom YOLOv8** - เมื่อต้องการ accuracy สูงสุดหรือ scale up

> [!TIP]
> เริ่มจาก GPT-4o เพื่อ validate idea และ collect data ก่อน แล้วค่อยลงทุน training custom model ภายหลัง
