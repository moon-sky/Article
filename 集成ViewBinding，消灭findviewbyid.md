## 集成ViewBinding，消灭findviewbyid



### Why？为什么要使用ViewBinding

```java
//case1        ViewGroup contentView = (ViewGroup) LayoutInflater.from(mContext).inflate(R.layout.dialog_private_custom, null);
        DialogPrivateCustomBinding dialogPrivateCustomBinding = DialogPrivateCustomBinding.inflate(LayoutInflater.from(mContext));

        String data = SpeechDialogConfigManager.instance().getCustomXiaoPData();
        if (!TextUtils.isEmpty(data)) {
            try {
                JSONObject json = new JSONObject(data);
                dialogPrivateCustomBinding.tvTitle.setText(json.optString("title", mContext.getString(R.string.st_custom_title)));
//case2                ((XTextView) contentView.findViewById(R.id.tv_title)).setText(json.optString("title", mContext.getString(R.string.st_custom_title)));
//                XTextView tv_custom_content = contentView.findViewById(R.id.tv_custom_content);
                dialogPrivateCustomBinding.tvCustomContent.setText(json.optString("content"));
                SpannableString span = new SpannableString(json.optString("guide"));
                span.setSpan(new ForegroundColorSpan(mContext.getColor(R.color.color_guide_content)), json.optInt("highlight_start"), json.optInt("hightlight_end"), Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
//                TextView tv_custom_guide = contentView.findViewById(R.id.tv_custom_guide);
                dialogPrivateCustomBinding.tvCustomGuide.setText(span);
//                ((XImageView) contentView.findViewById(R.id.iv_cover_one)).setImageResource(getResources().getIdentifier(json.optString("img", "icon_custom_xiaop"), "drawable", mContext.getPackageName()));
                dialogPrivateCustomBinding.ivCoverOne.setImageResource(getResources().getIdentifier(json.optString("img", "icon_custom_xiaop"), "drawable", mContext.getPackageName()));
```

我们针对上面代码中的case进行对比

Case1:相比传统的inflateview的方式，viewbinding更加简单，免去了输入contentview id的麻烦

Case2:省去findviewbyid这一步，直接减少50%代码量，另外viewbinding是类型安全的，我们不用强制类型转换，也很方便

### How？怎么使用

### 总结