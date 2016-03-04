(function ($) {
    var optimalHeadlineDatabase = Array();
    var phpDATA = optimal_headlines_data;

    optimalHeadlineGetHeadlines = function () {
        var headlines = Array();
        //regex to split on line ending
        headlines = phpDATA.headlines;

        return headlines;
    }

    // Get random with range
    optimalHeadlineRandom = function (min, max) {
        return Math.floor(Math.random() * (max - min + 1) + min);
    }

    // Display headlines
    optimalHeadlineDisplayHeadlines = function (headlines) {
        if (headlines.length > 0) {
            headline = headlines[0];

            headlines.splice(0, 1);
            jQuery('#optimal-headline-items').append('<li style="display:none">' + headline +
            '</li>').find('li:last-child').fadeIn('slow', function () {
                optimalHeadlineDisplayHeadlines(headlines);
            });
        }
    }
    jQuery(document).ready(function ($) {
        // Move meta box above editor
        $('#optimal-headline-meta-box').insertBefore('#postdivrich');

        // Generate headlines
        $('#optimal-headline-generate').click(function () {
            var topic = $('#optimal-headline-topic').val();

            $('#optimal-headline-items').slideUp('fast', function () {
                $('#optimal-headline-items li').remove();
            });

            if (optimalHeadlineDatabase.length < 8) {
                optimalHeadlineDatabase = Array();
                optimalHeadlineDatabase = optimalHeadlineGetHeadlines();
            }

            var i = 0;
            var headlines = Array();
            var headline = '';
            var randIndex = 0;

            for (i; i < 8; i++) {
                randIndex = optimalHeadlineRandom(0, optimalHeadlineDatabase.length);
                headline = optimalHeadlineDatabase[randIndex];
                headline = headline.replace('***', topic);
                headlines.push(headline);
                optimalHeadlineDatabase.splice(randIndex, 1);
            }

            optimalHeadlineDisplayHeadlines(headlines);

            $('#optimal-headline-items').slideDown('fast');

            return false;
        });

        // Populate Headlines
        $('#optimal-headline-items-wrap').on('click', '#optimal-headline-items li', function () {
            $('#title').val($(this).text());
            $('#title-prompt-text').addClass('screen-reader-text');
            $("html, body").animate({scrollTop: 0}, "fast");

        });

        // Reset Database
        $('#optimal-headline-topic').change(
            function () {
                optimalHeadlineDatabase = Array();
            }
        );

        // Enter Trigger Generate Button
        $('#optimal-headline-topic').keypress(function (e) {
            var key = e.which;
            // the enter key code
            if (key == 13) {
                $('#optimal-headline-generate').click();
                return false;
            }
        });
    });
})(jQuery);