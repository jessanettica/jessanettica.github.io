---
layout: post
title:  "Recreating Pinterest with Cats"
date:   2017-07-18 08:08:28 -0800
categories: python
image_sliders:
  - slider9
style: |
  .post-title {
    font-family: 'Playfair Display', serif;
    font-size: 45px;
  }
  .post-content p {
    font-family: 'Pontano Sans', sans-serif;
    font-size: 20px;
  }
  pre, code {
    background-color: #ffd7d7;
  }
photos: ["img/ocean.jpg", "img/beach.jpg", "img/cacti.jpg"]
---

{% include slider.html selector="slider9" %}

I’m not sure if Pinterest was the first website to popularize the cascading grid layout, but now I see it everywhere. If you’ve never used Pinterest, it’s a site where you can collect data-rich images that are displayed in a grid.

There are a couple of Cascading grid layout libraries are available and will handle the positioning for you (like [Masonry][masonry]), but implementing a Pinterest-like grid layout on your own is not that difficult. You can do it!

I’m going to use a bit of jQuery to explain how to do it, but use whatever you like. What matters is the math behind the positioning, not the front-end framework or library you use.  I’ll give you a general overview and then we can get into the implementation details with some code.

Let’s get into the general overview. The pins are loaded to the page on initial page load with AJAX. After assigning the column width and the margin between the columns, you can use the window width to calculate the number of columns that will fit on the page. You use an array to store the height of each column, and the array length is how many columns there are. For example,` myColumns = [300, 340, 370]` means that there are 3 columns on the page. The first column’s height is 300, second column’s height is 340, and the third column’s height is 370. Cool, right? So then when each pin is added to the page, it is added to the shortest column and the array is updated with the new column height. The left and top position of each of the absolutely-positioned pins is determined using these height values (see `getTopPosition()` and `getLeftPosition()` below).

That’s it! It is probably a bit confusing to just read that explanation so let’s take a look at the code to see how it’s actually implemented. I added comments with numbers so that you can read the file in the order it happens in the explanation above.

{% highlight javascript %}
function makeArray(len, value) {
  return new Array(len).fill(value);
}

function getShortestColumnIndex(colArray) {
  // returns which column is the shortest so that
  // we know where the next pin is gonna go
  return colArray.indexOf(Math.min(...colArray));
}

function getLeftPosition(colIndex, numColumns) {
  // calculate the left position of the pin taking
  // into account its width and what column it's in
  var gutter = 12;
  var pinWidth = 236;
  if (colIndex == 0){
    return gutter;
  }else{
    return gutter * (colIndex + 1) + pinWidth * colIndex;
  }
}

function getTopPosition(colIndex, colArray){
    // get the position of the top of the pin using
    // the height of the column it will be placed in
    // using the handy array we made
    return colArray[colIndex];
}

function updateColArray(colIndex, colArray, pinHeight){
  // update the column array by adding the pin height plus
  // the vertical space we want between pins
  colArray[colIndex] += (pinHeight + 120);
  return colArray;
}
var screenWidth = $(window).width();
var gutter = 12;
var numColumns = Math.floor(screenWidth / (236 + gutter));
// 2. Make an array to represent our columns and
// initialize height of all page columns to zero
var colArray = makeArray(numColumns, 0);

function renderPins(resp){
  // 3. Create DOM elements from the response data
  Object.values(resp["data"]).map(function(v) {
      return $(`<div data-height="${v.img_height}" style="padding:15px;position:absolute;">
        <img style="border-radius:10px;" width=236 src="${v.img_url}">
        <div style="min-height:13px;width: 236px;position:relative;padding-left:8px;padding-right:8px;">
            <div style="position:relative;max-width:145px;">
                <p style="max-width: 180px;margin-top:0;padding:0;max-height:60px;overflow:hidden;text-overflow:ellipsis;">${v.description}
                </p>
            </div>
            <div style="position:absolute;right:0;top:1px;z-index:3;">
                <p style="color:#a7a7a7">${v.repin_count} repins
                </p>
            </div>
        </div>
        <div style="display:flex;position:relative;-webkit-box-align:center;padding-left:8px;padding-right:8px;margin-top:4px;"
        >
            <div style="padding:0;display:flex;">
                <a href="pinterest.com/${v.pinner.username}" page style="-webkit-box-align: center;text-decoration:none">
                    <div style="height:24px;width:24px;margin-right:8px;">
                        <img src="${v.pinner.img_url}" style="height:24px;width:24px;border-radius:50%;position:static;">
                    </div>
                    <div>
                        <div style="display:block;overflow:hidden; text-overflow:ellipsis;">
                    ${v.pinner.username}</div>
                        <div style="display:block;overflow:hidden; text-overflow:ellipsis;">
                    ${v.boards.name}</div>
                    </div>
                </a>
             </div>
        </div>
      </div>`)
  }).forEach(function(v) {
      // get each pin image's height
      var imageHeight = v.data("height");
      var pinHeight = imageHeight + 24 + 24;
      // 4. Get the shortest column indext. The shortest column
      // is the one where the next pin will be placed.
      var columnIdx = getShortestColumnIndex(colArray);
      var left = getLeftPosition(columnIdx, numColumns);
      var top = getTopPosition(columnIdx, colArray);
      // 5. Update the array that represents our columns so that
      // we can continue putting future pins in the shortest column
      colArray = updateColArray(columnIdx, colArray, pinHeight);
      // 6. Add position to the pin
      v.css("top", top.toString() + "px");
      v.css("left", left.toString() + "px");
      v.css("width", "236px");
      v.css("height", "100%");
      // 7. Append the pin to the page
      $('#pin-container').append(v);
  });
}
function fetchAllInitialPins(){
  $.ajax({
      url: '/pins/get_all_pins/',
      type: 'POST',
      data: {},
      success: function(resp) {
          if (resp.status == 'ok'){
              renderPins(resp);
          }
      }
   });
}

window.onload = function () {
  // 1. Call the functions when the page loads
  fetchAllInitialPins();
};
{% endhighlight %}
One final note. The Ajax calls a function that is in the Django backend. Depending on what your stack is, it will be different, but this is what the function looks like:
{% highlight python %}
def get_all_pins(request):
    """Return http response with all pins to display on initial page load."""
    try:
        pins = Pin.objects.all()
        dictionaries = [ obj.as_dict() for obj in pins ]
    except:
        return json_response({
            'status': 'fail'
        })
    return json_response({
        'status': 'ok',
        'data': dictionaries
    })
{% endhighlight %}

If you want to look at the whole project implementation it’s [here][recreate-pinterest].


Jessica 👋

[masonry]: https://masonry.desandro.com/
[recreate-pinterest]: https://github.com/jessanettica/Recreate-Pinterest-with-cats
