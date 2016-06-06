title: Android 中关于视频检索
date: 2016-03-11 11:43:52
tags:
- Android
- 视频检索
- bitmap
---
```bash
MediaMetadataRetriever mmr = new MediaMetadataRetriever();
    public List<VideoInfo> getVideoList() {
        List<VideoInfo> videoInfoList = new ArrayList<>();
        String[] thumbColumn = {
                MediaStore.Video.Thumbnails.DATA,
                MediaStore.Video.Thumbnails.VIDEO_ID
        };
        String[] mediaColumns = {
                MediaStore.Video.Media._ID,
                MediaStore.Video.Media.DATA,
                MediaStore.Video.Media.TITLE,
                MediaStore.Video.Media.MIME_TYPE,
                MediaStore.Video.Media.DISPLAY_NAME,
                MediaStore.Video.Media.DURATION,
                MediaStore.Video.Media.HEIGHT,
                MediaStore.Video.Media.WIDTH
        };
        Cursor cursor = mContext.getContentResolver().query(MediaStore.Video.Media.EXTERNAL_CONTENT_URI, mediaColumns, null, null, null);
        if (cursor == null) {
            ToastUtil.showMessage("没有找到相关视频文件");
            return null;
        }
        if (cursor.moveToFirst()) {
            do {
                VideoInfo videoInfo = new VideoInfo();
                int id = cursor.getInt(cursor.getColumnIndex(MediaStore.Video.Media._ID));
                Cursor thumbCursor = mContext.getContentResolver().query(MediaStore.Video.Thumbnails.EXTERNAL_CONTENT_URI
                        , thumbColumn, MediaStore.Video.Thumbnails.VIDEO_ID + "=" + id, null, null);
                if (thumbCursor.moveToFirst()) {
                    String thumb = thumbCursor.getString(thumbCursor.getColumnIndex(MediaStore.Video.Thumbnails.DATA));
                    videoInfo.thumbPath = thumb;
                }
                videoInfo.id = id;
                    videoInfo.path = cursor.getString(cursor.getColumnIndexOrThrow(MediaStore.Video.Media.DATA));
                    videoInfo.title = cursor.getString(cursor.getColumnIndexOrThrow(MediaStore.Video.Media.TITLE));
                    videoInfo.duration = cursor.getLong(cursor.getColumnIndexOrThrow(MediaStore.Video.Media.DURATION));
                    mmr.setDataSource(videoInfo.path);
                    videoInfo.height = mmr.extractMetadata(MediaMetadataRetriever.METADATA_KEY_VIDEO_HEIGHT);
                    videoInfo.width = mmr.extractMetadata(MediaMetadataRetriever.METADATA_KEY_VIDEO_WIDTH);
                    videoInfoList.add(videoInfo);
            } while (cursor.moveToNext());
        }
        cursor.close();
        return videoInfoList;
    }
```
#存在的问题
##有些手机媒体哭保存视频列表会有缩略图而有些则没有。但是一我们呈现的是要有封面图也就是缩略图，那该如何获取缩略图呢？
我们可以这样，先判断手机里面是否有缩略图，也就是上面检索的结果是否有thumbCusor.getCounnt()`0;没有的话：
```bash
private void setThumb() {
        Task.callInBackground(new Callable<Object>() {
            @Override
            public Object call() throws Exception {
                getThumbBitmap();
                return null;
            }
        }).onSuccessTask(new Continuation<Object, Task<Object>>() {
            @Override
            public Task<Object> then(Task<Object> task) throws Exception {
                mVideoInfoAdapter.notifyDataSetChanged();
                return null;
            }
        });

    }

    //有些机型没有缩略图,需要生成bitmap
    private void getThumbBitmap() {
        for (VideoInfo mVideoInfo : mVideoInfoAdapter.getAll()) {
            if(TextUtil.isEmptyOrNull(mVideoInfo.thumbPath) || !new File(mVideoInfo.thumbPath).exists()){
                mVideoInfo.thumbBitmab = MediaStore.Video.Thumbnails.getThumbnail(getActivity().getContentResolver(), mVideoInfo.id, MediaStore.Video.Thumbnails.MINI_KIND, null);
            }
        }
    }

```
###存在的问题是：每次都要去生成缩略图，每次都很耗时 待优化；
参考文章：http://jayfeng.com/2016/03/16/%E7%90%86%E8%A7%A3ThumbnailUtils/
