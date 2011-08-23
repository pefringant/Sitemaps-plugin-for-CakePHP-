# What For !

This plugin alleviates some of the limits you may have encountered when using a custom controller to handle xml sitemaps according to the
norm specified here and which is implemented by web crawlers.
The first difficulty you may have met is the ressource-straining operation that represents the loading of many models from, for example, a SitemapsController.
Then, you may have noticed that some project may require that you adress the 50000 thousand ulrs limit set by the bots crawling your site. If you
have already solved those problems in your own way, maybe your collaboration on this plugin will help you and us in packaging the logic outside your application. Don't hesitate
to give us feedback !


We don't cover the creation of sitemap in the html format because the structure, appearance and content of such a document should be left to
the discretion of the developper, designer etc.

# How does it work ?

The plugin is based on a controller , a component that handles configuration, a helper to ease output of xml, and three views to adress the
different use cases we have found relevant.

First, the plugin builds a sitemapindex file containing the name of all the controllers you have thought to be worth sitemapping. After that,
each Controller and Model will be individually queried for the appropriate data in a separate php request made by the bot, saving most of
the ressources wasted in the case where you have to load all your models in the same request. More, the plugin's controller will decide through
its dispatch method if a classical sitemap has to be created or if there's a need to create a intermediary sitemapindex containing paginated urls. A great product catalog could
then be easily splitted in as many pages as you wish, since we give you the opportunity to set the number of urls you wish to see in each paginated
sitemap. Cake core caching technology is used to cache the sitemaps and the plugin gives you the possibility to set to your liking.

For more details, look at the source we tried to comment as exhaustively as possible.

# Getting to it

## Bate the bot
The first step to take is to create a robots.txt file if it doesn't exist in your project already. Then, add the following line
with your own settings.

Sitemap: http://your.domain.com/sitemaps/sitemaps/main.xml


## Configuring 
The "main" function of the Sitemaps.SitemapsController will take care of creating a list of controllers you want to be part of your xml
sitemap. Now, you may object that the creation of a sitemap is more the job for a model. Don't worry, we'll get to that. The sitemaps
plugin will look for a hardcoded array of controllers and their config values appended to your app/config/bootstrap.php in the following way

Configure::write ( 'Sitemap.Settings' ,  array (
                                  'controllers' =>
                                    array (
                                      'Stuffs'  =>
                                        array (
                                          'limit' => 2 ,
                                          'cacheDuration' => '1 Day'
                                        ) ,
                                      'Things'  =>
                                        array (
                                          'limit' => 3
                                        ) ,
                                      'Items' ,
                                      
                                    )
                                )
  ) ;

The parameter we allow you to set are :
  -- limit : although the bots allow 50000 urls per file, you may find that querying for 50000 records is going to be a problem for your
  application in terms of performance.
  -- cacheDuration : Use the syntax of the Cache helper to parametrize the time you want a bot query to be cached
  
## Sitemap Index ?  
  
The file generated by the Sitemaps.SitemapsController::main( ) function contents a list of controller Objects packed in the sitemapindex
format. Given the beforementioned example, it will look this way :

<sitemapindex xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
    <sitemap>
      <loc>
        http://your.domain.com/sitemaps/sitemaps/dispatch/stuffs.xml
      </loc>
    </sitemap>
    <sitemap>
      <loc>
        http://your.domain.com/sitemaps/sitemaps/dispatch/stuffs.xml
      </loc>
    </sitemap>
    <sitemap>
      <loc>
        http://your.domain.com/sitemaps/sitemaps/dispatch/things.xml
      </loc>
    </sitemap>
</sitemapindex>  
 
We have decided not to support the use of the <lastmod> node in the sitemapindex format because it required to query all the models 
    
  
Now, once we're set, let us talk about what the plugin requires from your part. Your application being a ____ mix of static and dynamic pages
that you may wish, or not, to be indexed, we give you complete freedom over the urls you feed the plugin.

## Static pages and modelless controllers.

Since you may want to let a page based on a controller without a model ( like it could be done in the case of a ContactController ) being indexed as well, some urls have to
be specified in the controller directly.The plugin requires the existence in each controller you've set in app/bootstrap.php of a function called
staticSitemap (  ) returning a set of flat urls. You can of course generate them wherever you want as long as the staticSitemap function returns
data as the plugin excepts it. We encourage of course to use the Router class to generate them. The function StuffsController::staticSitemap() would
look like this :

function staticSitemap() {
	  return array(
	      array(
		'loc' => Router::url(array('controller' => 'stuffs' , 'action' => 'add' , 'plugin' => null), true),
		'lastmod' => '2011-12-01',
		'changefreq' => 'yearly',
		'priority' => .5,
	      ),
	      array(
		'loc' => Router::url(array('controller' => 'stuffs' ,'action' => 'edit' , 'plugin' => null ), true),
		'lastmod' => '2012-12-01',
		'changefreq' => 'yearly',
		'priority' => .75,
	      ),
	  );
	}

The plugin will require a value for the 'loc' index and generate a fatal error if it's missing since <loc>is the only mandatory element of the sitemap protocol you have
to specify for each <url> node. To avoid any kind of trouble with routing, we strongly recommend you to set the plugin key to null.

## Dynamic pages
  
All the querying logic in the model is up to you. Two functions are needed here :
  Model::dynamicSitemapCount ( ) must return an integer count of records affected by the sitemap operations.
  Model::dynamicSitemap ( ) must return an array of urls formatted like they have to be in the Controller::staticSitemap ( ) method. Once
  again, the find logic is up to you, you may organize the content of those two methods the way you like it as long as the functions
  return the appropriate data. 

The Stuff model would look like this, with a very simple wrapper for your find query that could be conveniently cached since its going
to be used twice in a row by two successive executions of a php request.


    class Stuff extends AppModel {

      function dynamicSitemapCount ( ) {
  
    
  
      }

      function dynamicSitemap ( $offset = 0 , $limit = 50000 ) {
  
   
  
      }
  
  
    }


Then it's done. The plugin will count your static pages, your dynamic pages do the math and will either create a sitemap or a sitemapindex containing
paginated urls to query through the use of the $offset and $limit parameters of the dynamicSitemap methods.

## Misc

Although it seems a bit unlikely that the number of static pages could be greater than the number of dynamic pages, the limit you'll be setting
applies to both static and dynamic pages.

We can "only" generate the following number of dynamic urls :
 ( 50000 - number-of-static-pages ) * 50000
50000 * 50000 being equal to 25 000 000 000 urls, I think this limitation isn't the first you'll meet.

Since we cannot load automatically a routes.php file specific to the plugin, you'll have to add the following line to app/config/routes.php

Router::parseExtensions ( 'xml' )

or add the 'xml' value to the parameters of the alread called static function.



        


## Error Handling

We tried to make a smart mix between helping you out and require from you that you set your data coherently. However, your feedback would be
very appreciated to help us fining down this aspect of things if you found our way to be impedimenting your work.