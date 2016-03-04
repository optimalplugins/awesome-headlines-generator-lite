<?php
/*
  Plugin Name: Awesome Headlines Generator Lite
  Plugin URI: http://www.OptimalPlugins.com/
  Description: STOP Writing Headlines The Hard Way! Write SEO Friendly Headline Every Time - With This One Little Trick!
  Version: 1.0
  Author: OptimalPlugins.com
  Author URI: http://www.OptimalPlugins.com/
  License: GPLv2 or later
*/
if ( ! defined( 'ABSPATH' ) ) exit;

if(!class_exists('OptimalHeadline')):
final class OptimalHeadline {
    private static $instance;
    private $dir, $url, $table;
    const DS = DIRECTORY_SEPARATOR;
    
    public static function getInstance(){
        if(self::$instance==null){
            global $wpdb;
            self::$instance = new self;
            self::$instance->dir = dirname(__FILE__);
            self::$instance->url = WP_PLUGIN_URL . DIRECTORY_SEPARATOR . basename(self::$instance->dir);
            self::$instance->table = $wpdb->prefix . 'optimal_headline_data';
            require_once self::$instance->dir . '/headline_settings_page.php';
            require_once self::$instance->dir . '/headline_list.php';
            self::$instance->actions();
        }
        
        return self::$instance;
    }
    
    private function __construct() {
        ;
    }
    
    /**
     * call all actions/filters here
     */
    private function actions() {
        register_activation_hook(__FILE__, array($this,'activate'));
        register_deactivation_hook(__FILE__, array($this,'deactivate'));
        if(is_admin()) {
            add_action( 'admin_menu', array( $this, 'admin_menu' ) );

            add_action('admin_enqueue_scripts', array( $this, 'optimal_headline_init') );
            add_action('admin_menu', array( $this, 'optimal_headline_create_meta_box' ) );
            
            add_filter( 'plugin_action_links_' . plugin_basename(__FILE__), array( $this, 'add_action_links' ) );

            add_action("wp_before_admin_bar_render", array($this, 'adminBarCustom'));
        }
        
    }
    
    public function activate() {
        global $wpdb;
        $sqls = array();
        
        if (!empty ($wpdb->charset))
            $charset_collate = "DEFAULT CHARACTER SET {$wpdb->charset}";
        if (!empty ($wpdb->collate))
            $charset_collate .= " COLLATE {$wpdb->collate}";
        
        $sqls[] = "CREATE TABLE `{$this->table}` (
            `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
            `title` mediumtext COLLATE utf8_unicode_ci NOT NULL,
            `date` int(11) NOT NULL,
            `status` tinyint(1) NOT NULL DEFAULT '1',
            PRIMARY KEY `id` (`id`)
        );";    
            
        require_once(ABSPATH . 'wp-admin/includes/upgrade.php');
        foreach ($sqls as $key => $sql){
            dbDelta($sql);
        }
        
        //check for data 
        $query = "SELECT count(*) as cnt FROM {$this->table}";
        $results = $wpdb->get_row($query,ARRAY_A);
        if($results['cnt'] === "0") {
            //insert rows from files
            require_once $this->dir . '/headline_data.php';
            if(isset($optimal_headline_data) && is_array($optimal_headline_data)) {
                $query = "INSERT INTO {$this->table} (title,`date`) VALUES ";
                $cnt = count($optimal_headline_data);
                $i = 0;
                while ($i < $cnt) {
                    $time = time();
                    $title = esc_sql($optimal_headline_data[$i]);
                    $query .= "('{$title}',{$time})";
                    $i++;
                    if($i < $cnt)
                        $query .= ', ';
                }
                $wpdb->query($query);
            }
        }
    }
    
    public function deactivate() {
        
    }
    
    function add_action_links($links) {
/*
        $mylinks = array(
            '<a href="' . admin_url('options-general.php?page=optimal-headlines') . '">Settings</a>',
        );
        return array_merge($links, $mylinks);
*/
        return $links;
    }

    public function optimal_headline_init() {
        wp_register_style('optimal-headline-admin', plugins_url('/css/optimal-headline-admin.css', __FILE__));
        wp_enqueue_style('optimal-headline-admin');
        wp_register_script('optimal-headline-admin', plugins_url('/js/optimal-headline-admin.js', __FILE__));
        wp_enqueue_script('optimal-headline-admin');

        OptimalHeadline::set_headline_data();
    }

    public function set_headline_data() {
        global $wpdb;
        $query = "SELECT `title` FROM `{$this->table}` WHERE `status`=1";
        $results = $wpdb->get_col($query,0);
        wp_localize_script('optimal-headline-admin', 'optimal_headlines_data', array(
            'headlines' => $results
        ));
    }

    public function optimal_headline_create_meta_box() {
        $post_type_list = array('post', 'page');

        foreach ($post_type_list as $post_type) {
            add_meta_box('optimal-headline-meta-box',
                'Awesome Headlines Generator',
                array( $this, 'optimal_headline_meta_box' ),
                $post_type,
                'advanced',
                'high');
        }
    }

    public function optimal_headline_meta_box() {
        ob_start();
        ?>
        <table class="form-table">
            <tbody>
            <tr>
                <th class="optimal-headline-col1" scope="row"><label for="optimal-headline-topic">Headline about</label>
                </th>
                <td class="optimal-headline-col2"><input id="optimal-headline-topic"
                                                         placeholder="Type in your topic and click generate" value=""
                                                         name="optimal-headline" type="text"></td>
                <td class="optimal-headline-col3"><a id="optimal-headline-generate" href="#" class="button">Generate</a>
                </td>
            </tr>
            <tr>
                <th class="optimal-headline-col1" scope="row"></th>
                <td class="optimal-headline-col2" style="padding-top:0px">
                    <div id="optimal-headline-items-wrap">
                        <ul id="optimal-headline-items"></ul>
                    </div>
                </td>
                <td class="optimal-headline-col3">&nbsp;</td>
            </tr>
            </tbody>
        </table>
        <?php
        $html = ob_get_contents();
        ob_end_clean();

        echo $html;
    }
    
    public function admin_menu() {
    }
    
    public function settings_page() {
        OptimalHeadlineSettingsPage::getInstance();
    }

    public function adminBarCustom(){
        if(function_exists('get_current_screen')){
            $current_screen = get_current_screen();
            if($current_screen->id == 'settings_page_optimal-headlines'){
                require_once(self::$instance->dir . '/lib/banner/optimal-banner.php');
            }
        }
    }
}

OptimalHeadline::getInstance();
endif;
?>