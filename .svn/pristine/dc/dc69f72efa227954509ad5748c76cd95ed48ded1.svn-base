<?php

if ( ! defined( 'ABSPATH' ) ) exit;

if(!class_exists('WP_List_Table')){
    require_once( ABSPATH . 'wp-admin/includes/class-wp-list-table.php' );
}

class OptimalHeadlineList extends \WP_List_Table{
    private $table;
    function __construct(){
        global $wpdb;
        $this->table = $wpdb->prefix . 'optimal_headline_data';
        //Set parent defaults
        parent::__construct( array(
            'singular'  => 'headline',     //singular name of the listed records
            'plural'    => 'headlines',    //plural name of the listed records
            'ajax'      => false        //does this table support ajax?
        ) );
        
        
    }
    
    function get_columns(){
        $columns = array(
            'cb'            => '<input type="checkbox" />', //Render a checkbox instead of text
            'title'         => 'Headlines',
            'date'          => 'Date'
        );
        return $columns;
    }
    
    function column_cb($item){
        //echo $item['id'];
        //echo $column_name;
        return sprintf('<input type="checkbox" name="%s[]" value="%s" />',$this->_args['singular'],$item->id );
    }
    
    function column_default($item, $column_name){
        switch($column_name){
            case 'title':
                //Build row actions
                $actions = array(
                    'edit'      => sprintf('<a href="?page=%s&edit=%s" class="edit">Edit</a>',$_REQUEST['page'],$item->id),
                    'delete'    => sprintf('<a href="?page=%s&delete=%s" class="delete">Delete</a>',$_REQUEST['page'],$item->id),
                );

                //Return the title contents
                return sprintf('%1$s %2$s',
                    /*$1%s*/ $item->title,
                    /*$2%s*/ $this->row_actions($actions)
                );
                
            case 'date':
                return date('Y-m-d', $item->date);
                
            case 'active':
                $ret = "";
                if($item->active==="1"){
                    $ret = "Active";
                }
                elseif($item->active==="0"){
                    $ret = "In-active";
                }
                return sprintf('%s',$ret);
        }
    }
    
    function get_sortable_columns() {
        $sortable_columns = array(
            'title'     => array('title',false),     //true means it's already sorted
            'date'    => array('date',false),
        );
        return $sortable_columns;
    }
    
    function get_bulk_actions() {
        $actions = array(
            'bulk_edit' => 'Edit',
            'delete'    => 'Delete'
        );
        return $actions;
    }
    
    function process_bulk_action() {
        //echo "<pre>"; print_r($_GET); echo "</pre>";
        //Detect when a bulk action is being triggered...
        if( 'delete'===$this->current_action() && isset($_GET['headline'])) {
            global $wpdb;
           
            $cnt = count($_GET['headline']);
            $query = sprintf("DELETE FROM `{$this->table}` WHERE `id` in (%s)", implode(',', $_GET['headline']));
            $count = $wpdb->query($query);
            if( $count ) {
                echo "<div id='message' class='updated fade'><p><strong>$count Headlines Deleted Successfully.</strong></p></div>";
            } else {
                echo "<div id='message' class='updated fade'><p><strong>Something went wrong!!!</strong></p></div>";
                $wpdb->show_errors();
                $wpdb->print_error();
            }
            
        } 
        
    }
    
    function extra_tablenav( $which ) {
	if ( $which == "top" ){
           
	}
	if ( $which == "bottom" ){
		//The code that goes after the table is there
		//echo"Hi, I'm after the table";
	}
    }
    
    function prepare_items() {
	global $wpdb;
        
        $this->process_bulk_action();

        /* -- Preparing your query -- */
        $query = "SELECT * FROM `{$this->table}` WHERE `status`=1";

        /* -- Search Parameters -- */
        
        
        /* -- Order by Parameters --*/
        $orderby = (!empty($_REQUEST['orderby'])) ? esc_sql($_REQUEST['orderby']) : 'date'; //If no sort, default to title
        $order = (!empty($_REQUEST['order'])) ? esc_sql($_REQUEST['order']) : 'desc'; //If no order, default to desc
        
        if($orderby != '') {
            $query .= ' ORDER BY ' . $orderby . ' ' . $order;
        }
        
        
        /* -- Pagination parameters -- */
        $totalitems = $wpdb->query($query); //return the total number of affected rows
        $perpage = 20;
        $paged = !empty($_GET["paged"]) ? stripslashes($_GET["paged"]) : '';
        if(empty($paged) || !is_numeric($paged) || $paged<=0 ){ $paged=1; }
        $totalpages = ceil($totalitems/$perpage);
        if(!empty($paged) && !empty($perpage)){
                $offset=($paged-1)*$perpage;
            $query.=' LIMIT '.(int)$offset.','.(int)$perpage;
        }
        

        /* -- Register the Columns -- */
        $columns = $this->get_columns();
        $hidden = array();
        $sortable = $this->get_sortable_columns();
        $this->_column_headers = array($columns, $hidden, $sortable);
        
	/* -- Register the pagination -- */
        $this->set_pagination_args( array(
                "total_items" => $totalitems,
                "total_pages" => $totalpages,
                "per_page" => $perpage,
        ) );
        //echo $query;
	/* -- Fetch the items -- */
        $this->items = $wpdb->get_results($query);
    }
    
    
}
