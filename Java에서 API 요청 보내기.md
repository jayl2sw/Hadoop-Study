# Java에서 API 요청 보내기

```java
// image 파일을 byte 자료형으로 변환 
private static String base64EncodeFromFile(String imgurl) throws Exception {
    InputStream is = new URL(imgurl).openStream();
    byte[] bytes = IOUtils.toByteArray(is);
    String base64img = Base64.getEncoder().encodeToString(bytes);
    return base64img;
}
```

```java
public static String sendPostRequest(String urlString, JSONObject data) throws Exception {
    URL url = new URL(urlString);
    // HTTP 커넥션 열고
    HttpURLConnection con = (HttpURLConnection) url.openConnection();

    // HTTP Connection Setting
    con.setDoOutput(true);
    con.setDoInput(true);
    con.setRequestMethod("POST");
    con.setRequestProperty("Content-Type", "application/json");

    OutputStream os = con.getOutputStream();
    os.write(data.toString().getBytes());
    os.close();

    // 요청 보내고 InputStream 받음
    InputStream is = con.getInputStream();
    // inputstream -> byte -> String
    byte[] bytes = IOUtils.toByteArray(is);
    String response = new String(bytes);

    con.disconnect();
    return response;
}
```

```java
 public String sendAPIRequest(String path) throws Exception{

     // read image from local file system and encode
     JSONObject data = new JSONObject();
     data.put("api_key", apiKey);

     // add images
     JSONArray images = new JSONArray();
     String fileData = base64EncodeFromFile(path);
     images.put(fileData);

     data.put("images", images);

     // add modifiers (api 요청 설정)
     // modifiers info: https://github.com/flowerchecker/Plant-id-API/wiki/Modifiers
     JSONArray modifiers = new JSONArray()
         .put("crops_fast")
         .put("similar_images");
     data.put("modifiers", modifiers);

     // add language
     data.put("plant_language", "ko");

     // add plant details
     // more info here: https://github.com/flowerchecker/Plant-id-API/wiki/Plant-details
     JSONArray plantDetails = new JSONArray()
         .put("common_names")
         .put("url")
         .put("name_authority")
         .put("wiki_description")
         .put("taxonomy")
         .put("synonyms");
     data.put("plant_details", plantDetails);

     return sendPostRequest("https://api.plant.id/v2/identify", data);
 }
```



### 아쉬운점 

```java
public Map<String, Object> getGenusAndProb(String data) throws JSONException {
    Map<String, Object> res = new HashMap<>();
    JSONObject jsonObject = new JSONObject(data);
    JSONArray suggestions = (JSONArray) jsonObject.get("suggestions");
    JSONObject candidate = (JSONObject) suggestions.get(0);
    double probability = (double) candidate.get("probability");
    JSONObject plant_details = (JSONObject) candidate.get("plant_details");
    JSONObject structured_name = (JSONObject) plant_details.get("structured_name");
    String genus = (String) structured_name.get("genus");
    if (structured_name.has("species")) {
        String species = (String) structured_name.get("species");
        res.put("species", species);
    }

    res.put("probability", probability);
    res.put("genus", genus);


    return res;
}
```

key-value값에서 파싱을 한번하면 value 안에 있는 JSONObject가 String으로 나와서 또 해당 Object를 다시 파싱해주어야 한다.한번에 파싱 하는 방법이 없을까?