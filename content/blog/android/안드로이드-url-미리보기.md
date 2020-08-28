---
title: 안드로이드-URL-미리보기
date: 2020-08-28 10:08:03
category: android
thumbnail: './images/안드로이드_URL_미리보기.png'
draft: false
---

![](./images/안드로이드_URL_미리보기.png)

# 안드로이드 URL 미리보기 

> 설치하기 
> 
```sh
implementation "org.jsoup:jsoup:1.11.3"
```

> 중간에 이미지 프리뷰 대신 유투브 동영상 플레이어 넣고 샢을때 
```sh
implementation "com.pierfrancescosoffritti.androidyoutubeplayer:core:10.0.5"
```

## URL 프리뷰 클래스 만들기

```kotlin
class JsoupAsyncTask internal constructor() :
    AsyncTask<String?, JsoupEntity?, JsoupEntity?>() {

    override fun doInBackground(vararg params: String?): JsoupEntity? {
        try {
            var url = params[0].toString()

            if(url.indexOf("http") == -1) { // 정규표현식 적용시 http 없는애 체크할시에 예외 처리
                url = "http://$url"
            }
            val con = Jsoup.connect(url)
            val doc: Document = con.get()
            val ogTags: Elements = doc.select("meta[property^=og:]")
            var title = ""
            var image = ""
            var des = ""
            var siteName = ""
            var youtubeUrl = "" // youtube 동영상 플레이어 재생을휘해
            if (ogTags.size <= 0) {
                return null
            }
            // 필요한 OGTag를 추려낸다
            for (i in 0 until ogTags.size) {
                val tag: Element = ogTags.get(i)
                val text: String = tag.attr("property")
                when (text) {
                    "og:title" -> {
                        title =  tag.attr("content")
                    }
                    "og:image" -> {
                        image =  tag.attr("content")

                        val checkUrl = image.indexOf("/i1.daumcdn.net/")
                        if(checkUrl != -1) {
                            image = "http:$image"
                        }
                    }
                    "og:description" -> {
                        des =  tag.attr("content")
                    }

                    "og:site_name" -> {
                        siteName = tag.attr("content")
                    }

                    "og:url" -> {
                        youtubeUrl = tag.attr("content")
                    }
                }
            }

            val host: String = URL(url).host
            if(siteName.isEmpty()) siteName = host // host 주소에서 favicon 가져오기
            return JsoupEntity(title, image,des, siteName, "http://$host/favicon.ico", youtubeUrl)
        } catch (e: IOException) {
            e.printStackTrace()
        }
        return null
    }


    override fun onPostExecute(result: JsoupEntity?) {
    
    }

}
```

## 적용해보기

```kotlin
fun renderUrlPreview(view: LinearLayout, contentText: String, context: Context, fragmentManager :FragmentManager, isYouTubePlayer: Boolean) {

    view.removeAllViews()

    val regUrlPreview = Patterns.WEB_URL.toRegex() // 기본 내장 패턴 적용

    val matchResultAll: Sequence<MatchResult> =
        regUrlPreview.findAll(contentText!!) // 적용된 패턴으로 찾기

    val urlList = mutableListOf<String?>()

    matchResultAll.map {
        urlList.add(it.groups[0]?.value)
    }.joinToString()


    urlList.map {
        val ll = LinearLayout(context) // LinearLayout에 addview해서 화면 구성 
        ll.id = View.generateViewId()
        val data =  JsoupAsyncTask().execute(it).get() // 루프 돌면서 해당 주소 정보 가져오기

        if(data !== null) {
            val urlPreviewFragment = UrlPreviewFragment(it, data?.title, data?.image, data?.des, data?.siteName, data?.favicon, data?.youTube, isYouTubePlayer) // 프래그먼트로 화면 구성하기

            fragmentManager.beginTransaction().replace(ll.id, urlPreviewFragment).commit()

            view.addView(ll)

            val param =
                ll.layoutParams as ViewGroup.MarginLayoutParams
            param.setMargins(0,  TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP , 20.0f , context.resources.displayMetrics).toInt(), 0, 0)
            ll.layoutParams = param // 미리보기 사이 간격 조정
        }

    }

}
```