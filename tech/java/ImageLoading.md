# 이미지 파일 불러오기

## **Java에서 이미지 불러오기**

### **1. ImageIO 사용 (가장 일반적)**
```java
import javax.imageio.ImageIO;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;

public class ImageLoader {
    public static BufferedImage loadImage(String filePath) {
        try {
            File file = new File(filePath);
            return ImageIO.read(file);
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }
    
    public static void main(String[] args) {
        BufferedImage image = loadImage("images/photo.jpg");
        if (image != null) {
            System.out.println("이미지 로드 성공: " + image.getWidth() + "x" + image.getHeight());
        }
    }
}
```

### **2. 클래스패스에서 이미지 불러오기**
```java
import javax.imageio.ImageIO;
import java.awt.image.BufferedImage;
import java.io.InputStream;

public class ClassPathImageLoader {
    public static BufferedImage loadFromClassPath(String resourcePath) {
        try (InputStream is = ClassPathImageLoader.class.getResourceAsStream(resourcePath)) {
            if (is != null) {
                return ImageIO.read(is);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }
    
    public static void main(String[] args) {
        // resources 폴더의 이미지
        BufferedImage image = loadFromClassPath("/images/icon.png");
    }
}
```

### **3. URL에서 이미지 불러오기**
```java
import javax.imageio.ImageIO;
import java.awt.image.BufferedImage;
import java.net.URL;

public class URLImageLoader {
    public static BufferedImage loadFromURL(String imageUrl) {
        try {
            URL url = new URL(imageUrl);
            return ImageIO.read(url);
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }
    
    public static void main(String[] args) {
        BufferedImage image = loadFromURL("https://example.com/image.jpg");
    }
}
```

## **Spring Boot에서 이미지 불러오기**

### **1. 정적 리소스로 제공**
```java
// src/main/resources/static/images/photo.jpg
// 웹에서 접근: http://localhost:8080/images/photo.jpg
```

### **2. Controller에서 이미지 응답**
```java
import org.springframework.core.io.Resource;
import org.springframework.core.io.ResourceLoader;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
public class ImageController {
    
    private final ResourceLoader resourceLoader;
    
    public ImageController(ResourceLoader resourceLoader) {
        this.resourceLoader = resourceLoader;
    }
    
    @GetMapping("/image/{filename}")
    public ResponseEntity<Resource> getImage(@PathVariable String filename) {
        Resource resource = resourceLoader.getResource("classpath:static/images/" + filename);
        
        if (resource.exists()) {
            return ResponseEntity.ok()
                .contentType(MediaType.IMAGE_JPEG)
                .body(resource);
        } else {
            return ResponseEntity.notFound().build();
        }
    }
}
```

### **3. MultipartFile로 업로드된 이미지 처리**
```java
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;
import javax.imageio.ImageIO;
import java.awt.image.BufferedImage;

@RestController
public class ImageUploadController {
    
    @PostMapping("/upload")
    public String uploadImage(@RequestParam("file") MultipartFile file) {
        try {
            BufferedImage image = ImageIO.read(file.getInputStream());
            // 이미지 처리 로직
            return "업로드 성공: " + image.getWidth() + "x" + image.getHeight();
        } catch (IOException e) {
            return "업로드 실패: " + e.getMessage();
        }
    }
}
```

## **HTML에서 이미지 불러오기**

### **1. 기본 img 태그**
```html
<!-- 로컬 파일 -->
<img src="images/photo.jpg" alt="사진">

<!-- 웹 URL -->
<img src="https://example.com/image.jpg" alt="웹 이미지">

<!-- 상대 경로 -->
<img src="../images/photo.jpg" alt="상위 폴더 이미지">
```

### **2. CSS로 배경 이미지**
```html
<style>
.background-image {
    background-image: url('images/background.jpg');
    background-size: cover;
    width: 300px;
    height: 200px;
}
</style>

<div class="background-image"></div>
```

## **JavaScript에서 이미지 불러오기**

### **1. 동적으로 이미지 생성**
```javascript
// 새 이미지 객체 생성
const img = new Image();
img.src = 'images/photo.jpg';
img.alt = '사진';

// 로드 완료 이벤트
img.onload = function() {
    console.log('이미지 로드 완료:', img.width + 'x' + img.height);
    document.body.appendChild(img);
};

// 에러 처리
img.onerror = function() {
    console.error('이미지 로드 실패');
};
```

### **2. Canvas에 이미지 그리기**
```javascript
const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');
const img = new Image();

img.onload = function() {
    ctx.drawImage(img, 0, 0);
};

img.src = 'images/photo.jpg';
```

## **파일 경로 규칙**

### **상대 경로**
- `./images/photo.jpg` - 현재 폴더의 images 폴더
- `../images/photo.jpg` - 상위 폴더의 images 폴더
- `images/photo.jpg` - 현재 폴더 기준 (./ 생략 가능)

### **절대 경로**
- `/images/photo.jpg` - 루트 폴더의 images 폴더
- `C:/Users/username/Pictures/photo.jpg` - 전체 경로

## **지원하는 이미지 형식**
- **JPEG/JPG**: 사진에 적합, 압축률 높음
- **PNG**: 투명도 지원, 무손실 압축
- **GIF**: 애니메이션 지원
- **WebP**: 구글 개발, 압축률 높음
- **BMP**: 비트맵, 용량 큼

## **주의사항**
1. **파일 존재 확인**: 파일이 실제로 존재하는지 확인
2. **권한 확인**: 파일 읽기 권한이 있는지 확인
3. **메모리 관리**: 큰 이미지는 메모리 사용량 주의
4. **에러 처리**: 파일이 없거나 손상된 경우 처리
5. **보안**: 사용자 입력으로 파일 경로를 받을 때 주의 