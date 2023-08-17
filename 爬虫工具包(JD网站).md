## 我写的一个爬取jd的图片和商品的一个工具类

### 商品的entity
```java
@Data
@AllArgsConstructor
public class Project {
    private String data_lazy_img;
    private String content;
    private Double price;
    private String shopName;
}
```
### 工具类

```java
public class FileUtil {
    static final String PATH = "D:\\360TEST\\";
    static final String JD_URL = "https://search.jd.com";
    static final String HTTPS = "https:";

    //下载图片的方法
    public static void downloadPicByUrl(String path, String url, String name) {
        try {
            URL URL = new URL(url);
            HttpURLConnection urlConnection = (HttpURLConnection) URL.openConnection();
            int responseCode = urlConnection.getResponseCode();
            if (responseCode == HttpURLConnection.HTTP_OK) {
                InputStream inputStream = urlConnection.getInputStream();
                BufferedInputStream bufferedInputStream = new BufferedInputStream(inputStream);
                FileOutputStream fileOutputStream = new FileOutputStream(path + name);
                BufferedOutputStream bufferedOutputStream = new BufferedOutputStream(fileOutputStream);
                byte[] bytes = new byte[1024];
                int len;

                while ((len = bufferedInputStream.read(bytes)) != -1) {
                    bufferedOutputStream.write(bytes, 0, len);
                }
                bufferedOutputStream.flush();
                bufferedInputStream.close();
                bufferedOutputStream.close();
                urlConnection.disconnect();
            }
        } catch (MalformedURLException e) {
            e.printStackTrace();
            throw new RuntimeException("url地址不对,图片下载失败");
        } catch (IOException e) {
            e.printStackTrace();
            throw new RuntimeException("文件传输出错");
        }
    }

    public static void downloadPicByUrl(String url) {
        FileUtil.downloadPicByUrl(PATH, url, System.currentTimeMillis() + ".jpg");
    }

    //爬虫网页图片并下载方法(京东)
    public static void webJDImageCrawler(String path, String keyWord) {
        Document document = null;
        String url = JD_URL + "/Search?keyword=" + keyWord;
        try {
            document = Jsoup.connect(url).get();
        } catch (IOException e) {
            e.printStackTrace();
            throw new RuntimeException("连接文档失败");
        }
        Elements imgElements = document.select("img");
        for (Element imgElement : imgElements) {
            String src = imgElement.attr("data-lazy-img");
            if (StrUtil.isBlank(src)) {
                continue;
            }
            FileUtil.downloadPicByUrl(path, HTTPS + src, System.currentTimeMillis() + ".jpg");
        }
    }

    public static void webJDImageCrawler(String url) {
        FileUtil.webJDImageCrawler(PATH, url);
    }

    //爬虫商品的信息并封装
    public static List<Project> crawlerJDProduct(String keyword, List<Integer> page) {
        //sku-name
        ArrayList<Project> projects = new ArrayList<>();

        for (Integer integer : page) {
            String clazz = "gl-item";
            String src = "https://search.jd.com/Search?keyword=" + keyword + "&page=" + integer;
            Document document = null;
            try {
                document = Jsoup.connect(src).get();
            } catch (IOException e) {
                e.printStackTrace();
                throw new RuntimeException("连接文档失败");
            }
            Elements select = document.select("." + clazz);
            for (Element element : select) {
                Elements img = element.select("img");
                //价格
                String price = element.select(".p-price").select("i").text();
                //获取图片
                String data_lazy_img = img.attr("data-lazy-img");
                //获取内容
                String content = element
                        .select(".p-name")
                        .select("em")
                        .text()
                        .replaceAll("<font class=\"skcolor_ljg\">", "")
                        .replaceAll("</font>", "");
                //获取店家
                String shopName = element
                        .select(".curr-shop")
                        .attr("title")
                        .replaceAll("<i data-price=\"(\\d+)\">", "");
                Project project = new Project(data_lazy_img, content, Double.parseDouble(price), shopName);
                projects.add(project);
            }
        }
        return projects;
    }
    //下载商品列表里面的所有图片到本地d:\\360TEST\\
    public static void downLoadPicByProductList(List<Project> projects){
        for (Project project : projects) {
            String data_lazy_img = project.getData_lazy_img();
            FileUtil.downloadPicByUrl(PATH, HTTPS + data_lazy_img, project.getShopName() +System.currentTimeMillis() + ".jpg");
        }
    }
}

```