---
layout: post
title:  "jquery-file-upload"
date:   2015-01-15
categories: blog
tags: [jquery] 
author: shobhit_garg
share: true
comments: true
excerpt:
---

Sometimes you need to provide a functionality to user to upload the file on server.There are many plugin on web to do this but i found [jquery-file-upload][jfu] nice and easy to use.This provides a lot of functionality but  here i am describing the basic functionality  which we all need generally.


Apart from jquery standard js file you need three more files which are:

* jquery.ui.widget.js
* jquery.iframe-transport.js
* jquery.fileupload.js

These files can be downloaded from this [location][link].

Now create a html file say file_uploader.html and include these files in the same order.

{% highlight ruby %}
#include main jquery js
<script src="/assets/shared/jquery.ui.widget.js"></script>
<script src="/assets/shared/jquery.iframe-transport.js"></script>
<script src="/assets/shared/jquery.fileupload.js"></script>
{% endhighlight %}

After that you have to add a input field for uploading the files.

{% highlight ruby %}
<input id="fileupload" type="file" name="files[]" data-url="/upload-file/" multiple>
#data-url is the server location where you want these files to upload when user tries to upload
{% endhighlight %}

Now for uploading files use this script:

{% highlight ruby %}
<div id="progress" style="position:absolute;width:100px;">
  <div class="bar" style="height:18px;background: green;width: 0%"></div>
</div>


<script>
$(function () {
        $('#fileupload').fileupload({
            dataType: 'json',
            add: function (e, data) {
                data.context = $('<p/>').text('Uploading...').appendTo(document.body);
                data.submit();
            },
            done: function (e, data) {
                data.context.text('Upload finished.');
                $.each(data.result.files, function (index, file) {
                    $('<p/>').text(file.name + ":"+ file.message).appendTo(document.body);
                });
            },

            progressall: function (e, data) {
                var progress = parseInt(data.loaded / data.total * 100, 10);
                $('#progress .bar').css(
                        'width',progress + '%'
                );
            }

        });
    });
</script>
{% endhighlight %}

At the start of this code a div for progress bar has added.As your files get uploaded this progress bar would display the progress.As we are providing green color to progress bar so make sure that your background color shouldn't be green.

Using this you can upload multiple files.In this case files get uploaded to the server one by one and response would be returned from server for each file.The data which server should return on uploading should be of format:

{% highlight ruby %}

{"files"=> [
        {
            "name" : "picture1.jpg", #filename
            "message" : "Uploaded Successfully."
        },
        {
            "name" : "picture2.png", #filename
            "message" : "Filetype not allowed."
        }
    ]}

{% endhighlight %}

Now say you want to upload all the files in one go for this you have to set _singleFileUploads_ to false. In many cases you would want to send some other data to server like some ids along with files.For this you have to set data in _formData_ .

{% highlight ruby %}
 $(function () {
        $('#fileupload').fileupload({
            dataType: 'json',
            singleFileUploads: false,
            formData: [
                {
                    name: 'a',
                    value: 1
                },
                {
                    name: 'b',
                    value: 2
                }
            ],
            add: function (e, data) {
                data.context = $('<p/>').text('Uploading...').appendTo(document.body);
                data.submit();
            },
            done: function (e, data) {
                data.context.text('Upload finished.');
                $.each(data.result.files, function (index, file) {
                    $('<p/>').text(file.name + ":"+ file.message).appendTo(document.body);
                });
            },

            progressall: function (e, data) {
                var progress = parseInt(data.loaded / data.total * 100, 10);
                $('#progress .bar').css(
                        'width',progress + '%'
                );
            }

        });
    });
{% endhighlight %}
Here remember,format of formData should remain same.On server this data would go as:

{% highlight ruby %}
{"a":1,"b":1,"files":[....]}
{% endhighlight %}

On backend side you have to read the files one by one from the array and do processing on it.For backend,technology can be different like php,rails,java etc. so you have to read the file according to functions provided by the respected technology.


For more options please check [this][options] out.

[jfu]: https://github.com/blueimp/jQuery-File-Upload
[link]: https://github.com/blueimp/jQuery-File-Upload/releases
[options]: https://github.com/blueimp/jQuery-File-Upload/wiki/Options
