---
layout: post
title: Limit the number of lines in a TextArea
summary: This limits the number of lines of any textarea that has the attribute 'data-limit-rows' set to 'true'.
---

This limits the number of lines of any textarea that has the attribute 'data-limit-rows' set to 'true'. The limit is the 'rows'-attribute of the textarea.

To know how many lines have been entered you have to look at the number of newline characters (\n) in the text. You could do this by splitting text on the newline character, and looking at the length of the resulting array: i.e. myText.split(/\n/g).length; This would work fine in every browser, except for one special case in Internet Explorer 8, whereby the user hits the enter key several times in a row, thus producing empty lines. In this case Internet Explorer 8 excludes all empty values from the resulting array, ie those places where the delimiters appear next to each other. The result is that the user can enter more lines of text than permitted. This is probably also true for Internet Explorer versions lower than 8, but I didn't check.

To know which key was pressed it's best to use jQuery's event.which as that means you don't have to bother with charCode and keyCode.

### The Code

{% highlight html %}
 <textarea data-limit-rows="true" cols="60" rows="8"></textarea>
{% endhighlight %}

{% highlight js %}
$(document).ready(function () {
  $('textarea[data-limit-rows=true]')
    .on('keypress', function (event) {
        var textarea = $(this),
            text = textarea.val(),
            numberOfLines = (text.match(/\n/g) || []).length + 1,
            maxRows = parseInt(textarea.attr('rows'));

        if (event.which === 13 && numberOfLines === maxRows ) {
          return false;
        }
    });
});
{% endhighlight %}