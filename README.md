# My Website AKA Questionable experiments in Quarto

This is a work in progress, please come back later :) 

The output/website can be accessed at: https://questionable.quarto.pub/blog/

## Inspiration 

There are so many awesome resources out there making building your own website/blog a breeze. Here are some of my favorites that I've stumbled across in my journey that you might enjoy: 

 - [The ultimate guide to starting a Quarto blog](https://albert-rapp.de/posts/13_quarto_blog_writing_guide/13_quarto_blog_writing_guide.html)
 - [Creating a blog with Quarto in 10 steps](https://beamilz.com/posts/2022-06-05-creating-a-blog-with-quarto/en/)
 - [Notes from a data witch](https://blog.djnavarro.net/posts/2022-04-20_porting-to-quarto/)
 - This is rmarkdown, but [Yihui's home page](https://yihui.org/todo/) is beautiful


## Previewing 

The website can be previewed by using the terminal to move into the main quarto website folder and then running `quarto preview`. 

## Make it pretty 

There are so many different [theme options](https://quarto.org/docs/output-formats/html-themes.html#overview) available to use. You can even have a different theme for [light and dark mode](https://quarto.org/docs/output-formats/html-themes.html#dark-mode) (perfect for anyone like me who is in love with the vapor theme but want to default to something a little less "loud"). 

Some of my unapologetic favorites are: 
 - [quartz](https://bootswatch.com/quartz/) (that gradient!)
 - [vapor](https://bootswatch.com/vapor/) (who isn't into vaporwave/cyberpunk)
 - [sketchy](https://bootswatch.com/sketchy/) (I've always wanted to be an artist)

I had some challenges trying to get the theme to use the secondary colors instead of using the primary everywhere. 

This is where [notes from a data witch](https://blog.djnavarro.net/posts/2022-04-20_porting-to-quarto/#styling-the-new-blog) is a legend. Using that method of creating a custom `.scss` file that copies itself from the theme of my choice. 

We do that by referencing our custom `.scss` file in our yaml with: 
    `theme: `
    `  light: custom_theme.scss`

From the about pages we can learn about the [different templates](https://quarto.org/docs/websites/website-about.html) that come built in to quarto for making it really easy to lay things out. 

## Publishing 

Done through: https://quarto.org/docs/publishing/quarto-pub.html 

I have two domains I can publish to: 

 - questionable.quarto.com
 - lisa.quarto.com
 
I run `quarto publish quarto-pub` after cd-ing in to my my_website directory. Answer "Y" to overwrite my previous site and to use the correct account. 

I can access my account and see my deployments at https://questionable.quarto.pub/blog/. 

TODO: Figure out how to overwrite only the changes instead of a completely new deployment (if possible). 

## Publishing 2.0 - now make it automated using github actions

TODO
 
## Troubleshooting

ojs errors: This was resolved by completely uninstalling quarto, uninstalling the RStudio IDE, and then re-installing. 
 
 
