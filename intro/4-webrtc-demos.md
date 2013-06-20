!SLIDE example1-slide small

# getUserMedia Demo

<video id="example1-video" autoplay="autoplay">

<script>
  $(".example1-slide").bind("showoff:show", function() {
    navigator.webkitGetUserMedia(
      {video: true, audio: false},
      function(stream) {
        document.getElementById('example1-video').src =
            webkitURL.createObjectURL(stream);
      }
    );
  });
</script>


!SLIDE small small-code

# getUserMedia Demo

       @@@ javascript
       <video id="example1-video" autoplay="autoplay">

       <script>
         navigator.webkitGetUserMedia(
           {video: true, audio: false},
           function(stream) {
             document.getElementById('example1-video').src =
                 webkitURL.createObjectURL(stream);
           }
         );
       </script>
