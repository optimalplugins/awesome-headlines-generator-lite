<?php

if ( ! defined( 'ABSPATH' ) ) exit;

final class OptimalHeadlineSettingsPage {
    private static $instance;
    private $error, $messages,$table;
    public static function getInstance(){
        if(self::$instance==null){
            self::$instance = new self;
            //
            self::$instance->actions();
        }
        
        return self::$instance;
    }
    
    private function __construct() {
        global $wpdb;
        $this->messages=array();
        $this->table = $wpdb->prefix . 'optimal_headline_data';
    }
    
    private function actions() {
        ob_start();
        if(isset($_REQUEST['add_new']) ){
            $this->add_new();
        }
        elseif (isset ($_REQUEST['bulk_import'])) {
            $this->bulkImport();
        }
        elseif (isset ($_REQUEST['edit'])) {
            $this->edit();
        }
        elseif (isset($_REQUEST['delete'])){
            $this->delete();
        }
        elseif(isset ($_GET['action'] , $_GET['headline']) && $_GET['action'] === "bulk_edit") {
            $this->bulkEdit();
        }
        else {
            $this->listTable();
            
        }
        $content = ob_get_clean();
        global $wpdb;
        //$wpdb->show_errors();
        //$wpdb->print_error();
        echo $this->get_header();
        echo $content;
        echo $this->get_footer();
        
    }
    
    private function bulkEdit() {
        if ( ! empty( $_POST ) && check_admin_referer( 'bulk_edit_headline_nonce', 'bulk_edit_headline' ) ) {
            //echo "<pre>"; print_r($_POST['title']); echo "</pre>";
            $titles   = ( isset( $_POST['title'] ) )   ?  $_POST['title']  : '';
            
            //validation
            $data = array();
            if(is_array($titles) && !empty($titles)) {
                foreach ($titles as $key => &$value) {
                    
                    $value = esc_attr( trim( $value ) );
                    $data[] = array('id' => (int) $key, 'title' => $value);
                    if($value == "") {
                        $this->error['title'][$key] = "Headline is required.";
                    }
                }
            } else {
                $this->messages[] = "Invalid Input";
            }
            
            if(empty($this->messages) && empty($this->error)) {
                //update to database
                global $wpdb;
                
                //update multiple row at a time
                
                $query = "UPDATE `{$this->table}` SET title= CASE id ";
                $ids = array_keys($titles);
                foreach ($titles as $key => $val) {
                    $query .= "WHEN {$key} THEN '{$val}' ";
                }
                $query .= " END WHERE id IN ( " . implode(',', $ids) .  ") ";
                
                if($wpdb->query($query) !== FALSE) {
                    $location = add_query_arg(array('page'=>'optimal-headlines','task_ok'=>'5'),  admin_url('options-general.php?'));
                    $this->javascript_redirect($location);
                } else {
                    $this->messages[] = "Somethis went wrong!!!";
                    $wpdb->show_errors();
                    $wpdb->print_error();
                    $this->bulkEditForm($data);
                }
            } else {
                $this->bulkEditForm($data);
            }
            
        } else {
            //get all data form database
            global $wpdb;
            $query = sprintf("SELECT id,title FROM `%s` WHERE `id` IN (%s)", $this->table, implode(',', $_GET['headline']));
            $data = $wpdb->get_results($query,ARRAY_A);
            $this->bulkEditForm($data);
        }
    }
    
    private function bulkEditForm($data = array()) {
        
        $nonce = wp_nonce_field( 'bulk_edit_headline_nonce' , 'bulk_edit_headline', true, false ); 
        ?>
        <form method="post" action="">
            <?php echo $nonce; ?>
            <table class="form-table">
                <tbody>
                    <tr class="form-field form-required">
                        <th scope="row" valign="top">
                            <label for="name">Headlines</label>
                        </th>
                        <td>
                        <?php if(!empty($data)): 
                            foreach ($data as $row):
                            ?>
                                <input name="title[<?php echo $row['id']; ?>]" type="text" aria-required="true" value="<?php echo $row['title'];   ?>" />
                                <p>&nbsp;</p>
                                <?php if(isset($this->error['title'][$row['id']])) echo '<p style="color:red;"><strong>' . $this->error['title'][$row['id']] . '</strong></p>';  ?>
                                <br />
                            <?php
                            endforeach;
                        endif; ?>
                            
                        </td>
                    </tr>
                </tbody>
            </table>

            <p class="submit">
                <input type="submit" name="submit" id="submit" class="button button-primary" value="Import"/>
                <?php $link = add_query_arg(array(),  admin_url('options-general.php?page=optimal-headlines'));  echo '<a href="' . esc_url($link) .'" class="button button-primary">Back</a>';  ?>
            </p>
                
        </form>

        <?php
    }
    
    private function bulkImport() {
        if ( ! empty( $_POST ) && check_admin_referer( 'bulk_import_headline_nonce', 'bulk_import_headline' ) ) {
            $titles   = ( isset( $_POST['titles'] ) )   ?  $_POST['titles']  : '';
            if(!$titles) {
                //check for validation 
                $this->error['titles'] = "This field cannot be empty.";
                $this->bulkImportForm();
            } else {
                global $wpdb;
                $title_array = explode("\n", str_replace("\r", "", $titles));
                if(isset($title_array) && is_array($title_array)) {
                    $query = "INSERT INTO {$this->table} (title,`date`) VALUES ";
                    $cnt = count($title_array);
                    $i = 0;
                    while ($i < $cnt) {
                        //check for empty string
                        if( trim( $title_array[$i] ) == "") {
                            $i++;
                            continue;
                        }
                        
                        $time = time();
                        $title = esc_sql( trim ($title_array[$i]) );
                        $query .= "('{$title}',{$time})";
                        $i++;
                        if($i < $cnt)
                            $query .= ', ';
                    }
                    if($wpdb->query($query)) {
                        $location = add_query_arg(array('page'=>'optimal-headlines','task_ok'=>'4'),  admin_url('options-general.php?'));
                        $this->javascript_redirect($location);
                    } else {
                        $this->messages[] = "Somethis went wrong!!!";
                        $wpdb->show_errors();
                        $wpdb->print_error();
                        $this->bulkImportForm(array('titles'=>$titles));
                    }
                }
            }
            
        } else {
            $this->bulkImportForm();
        }
    }
    
    private function bulkImportForm($data = array()) {
        $nonce = wp_nonce_field( 'bulk_import_headline_nonce' , 'bulk_import_headline', true, false ); 
        
        if(empty($data)){
            $data = array(
                "titles" => "",
            );
        }
        
        ?>
        <form method="post" action="">
            <?php echo $nonce; ?>
            <table class="form-table">
                <tbody>
                    <tr class="form-field form-required">
                        <th scope="row" valign="top">
                            <label for="name">Headline</label>
                        </th>
                        <td>
                            <textarea name="titles" id="title" rows="25" aria-required="true"><?php echo $data['titles'];   ?></textarea>
                            <p>Enter Your Headlines Here, One Headline Per line</p>
                            <?php if(isset($this->error['titles'])) echo '<p style="color:red;"><strong>' . $this->error['titles'] . '</strong></p>';  ?>
                        </td>
                    </tr>
                </tbody>
            </table>

            <p class="submit">
                <input type="submit" name="submit" id="submit" class="button button-primary" value="Import"/>
                <?php $link = add_query_arg(array(),  admin_url('options-general.php?page=optimal-headlines'));  echo '<a href="' . esc_url($link) .'" class="button button-primary">Back</a>';  ?>
            </p>
                
        </form>

        <?php
    }
    
    private function add_new() {
        $data = array();
        if ( ! empty( $_POST ) && check_admin_referer( 'create_headline_nonce', 'create_headline' ) ) {
            
            // Grab the submitted data
            $title   = ( isset( $_POST['title'] ) )   ? (string) stripslashes( $_POST['title'] ) : '';
            
            $active = ( isset( $_POST['active'] ) ) ? (int) $_POST['active'] : '';
            
            //check for validation 
            if(!$title){
                $this->error['title'] = "Please specify a headline";
            }
            
            
            if(empty($this->error) && empty($this->messages)){
                $data = array(
                    "title" => $title,
                    "date" => time(),
                );
                $format = array("%s","%d");
                
                global $wpdb;
                if($wpdb->insert($this->table,$data,$format)){
                    $location = add_query_arg(array('page'=>'optimal-headlines','task_ok'=>'1'),  admin_url('options-general.php?'));
                    $this->javascript_redirect($location);
                    
                } else {
                    $this->messages[] = "Somethis went wrong!!!";
                    $wpdb->show_errors();
                    $wpdb->print_error();
                }
            }
        }
        
        $this->addEdit($data);
        
    }
    
    private function edit(){
        global $wpdb;
        if ( ! empty( $_POST ) && check_admin_referer( 'edit_headline_nonce', 'edit_headline' ) ) {
            // Grab the submitted data
            $title   = ( isset( $_POST['title'] ) )   ? (string) stripslashes( $_POST['title'] ) : '';
            //check for validation 
            if(!$title){
                $this->error['title'] = "Please specify a headline";
            }
            
            if(empty($this->error) && empty($this->messages)) {
                $data = array (
                    "title" => $title,
                    "date" => time(),
                );
                $format = array("%s","%d");
                $where  = array('id' => (int)$_REQUEST['edit']);
                $format_where = array('%d');
                if($wpdb->update($this->table,$data,$where,$format,$format_where)){
                    $location = add_query_arg(array('page'=>'optimal-headlines','task_ok'=>'2'),  admin_url('options-general.php?'));
                    $this->javascript_redirect($location);
                }
                else {
                    $this->messages[] = "Something went wrong!!!";
                    $wpdb->show_errors();
                    $wpdb->print_error();
                    $this->addEdit($data);
                }
            }
        }
        else {
            $data = $wpdb->get_row($wpdb->prepare("SELECT * FROM `{$this->table}` WHERE id=%d", array($_REQUEST['edit'])),ARRAY_A);
            //print_r($data);
            if(empty($data) ) {
                $this->messages[] = "No records found with id {$_REQUEST['edit']}";
            }
            $this->addEdit($data);
            
        }
    }
    
    private function addEdit($data = array()){
        //print_r($data);
        if(isset($_REQUEST['edit']) && $_REQUEST['edit'] != ""){
            $nonce = wp_nonce_field( 'edit_headline_nonce' , 'edit_headline', true, false ); 
            $readonly = "readonly='readonly'";
            $hidden_field = '';
        }
        else {
            $nonce = wp_nonce_field( 'create_headline_nonce' , 'create_headline', true, false ); 
            $readonly = "";
            $hidden_field = "<input type='hidden' name='add_new' value='1'>";
        }
        
        if(empty($data)){
            $data = array(
                "title" => "",
            );
        }
    ?>
        <form method="post" action="">
            <?php echo $nonce; echo $hidden_field; ?>
            <table class="form-table">
                <tbody>
                    <tr class="form-field form-required">
                        <th scope="row" valign="top">
                            <label for="name">Headline</label>
                        </th>
                        <td>
                            <input name="title" id="title" type="text" aria-required="true" value="<?php echo $data['title'];   ?>" <?php //echo $readonly; ?> />
                            <p>Enter Your Headline Here</p>
                            <?php if(isset($this->error['title'])) echo '<p style="color:red;"><strong>' . $this->error['title'] . '</strong></p>';  ?>
                        </td>
                    </tr>
                </tbody>
            </table>

            <p class="submit">
                <input type="submit" name="submit" id="submit" class="button button-primary" value="<?php if(isset($_REQUEST['edit'])) { echo "Edit Headline"; } else { echo  "Add New Headline";} ?>"/>
                <?php $link = add_query_arg(array(),  admin_url('options-general.php?page=optimal-headlines'));  echo '<a href="' . esc_url($link) .'" class="button button-primary">Back</a>';  ?>
            </p>
                
        </form>

    <?php

    }
    
    private function delete(){
        if ( ! empty( $_POST ) && check_admin_referer( 'delete_headline_nonce', 'delete_headline' ) ) {
            $id = (int)$_GET['delete'];
            if($id > 0) {
                global $wpdb;
                $where = array('id' => $id);
                $where_format = array('%d');
                if( $wpdb->delete( $this->table, $where, $where_format ) ) {
                    $location = add_query_arg(array('page'=>'optimal-headlines','task_ok'=>'3'),  admin_url('options-general.php?'));
                    $this->javascript_redirect($location);
                } else {
                    $this->messages[] = "Something went wrong!!!";
                    $wpdb->show_errors();
                    $wpdb->print_error();
                    $this->addEdit($data);
                }
            } 
        } else {
            $this->deleteHtml($_REQUEST['delete']);
        }
    }
    
    private function deleteHtml($id) {
        $id = (int) $id;
        if($id > 0) {
            $nonce = wp_nonce_field( 'delete_headline_nonce' , 'delete_headline', true, false ); 
            ?>
        <form method="post">
            <?php echo $nonce; ?>
            Are you sure you want to delete this headlines ?
            <p class="submit">
            <input type="submit" name="submit" id="submit" class="button button-primary" value="Delete"/>
            <?php $link = add_query_arg(array(),  admin_url('options-general.php?page=optimal-headlines'));  echo '<a href="' . esc_url($link) .'" class="button button-primary">Back</a>';  ?>
            </p>
        </form>
            <?php
        }
    }
    
    
    
    private function listTable(){
        $table = new OptimalHeadlineList();
        //Fetch, prepare, sort, and filter our data...
        $table->prepare_items();
        ?>    
        <form id="movies-filter" method="get">
            <!-- For plugins, we also need to ensure that the form posts back to our current page -->
            <input type="hidden" name="page" value="<?php echo $_REQUEST['page'] ?>" />
            <!-- Now we can render the completed list table -->
            <?php $table->display() ?>
        </form>
        <?php    
    }
    
    
    
    private function get_header() {
        if(isset($_REQUEST['add_new']) && $_REQUEST['add_new'] != ""){
            $title = "Add New Headline";
            
        }
        elseif(isset ($_REQUEST['edit']) && $_REQUEST['edit'] != ""){
            $title = "Edit Headline";
        }
        else {
            $url = add_query_arg(array('add_new'=>'1'),  admin_url('options-general.php?page=optimal-headlines'));
            $link = '<a href="' . esc_url($url) .'" class="add-new-h2">Add New</a>';
            
            $url2 = add_query_arg(array('bulk_import'=>'1'),  admin_url('options-general.php?page=optimal-headlines'));
            $link2 = '<a href="' . esc_url($url2) .'" class="add-new-h2">Bulk Import</a>';
            $title = "Headlines List " . $link . $link2;
            
        }
    ?>
        <div class="wrap">
                <div id="icon-edit" class="icon32 icon32-posts-post">&nbsp;</div>
                <h2><?php echo $title; ?></h2>
                <br class="clear">
    <?php
        
        $str = '';
        if(!empty($this->messages)){
            $str .= "<div id='message' class='updated fade'>";
            foreach ($this->messages as $key => $msg):
                $str .= "$msg <br>";
            endforeach;
            $str .= "</div>";
            echo $str;
        }
        if(isset($_REQUEST['task_ok'])){
            if($_REQUEST['task_ok']==1){
                echo "<div id='message' class='updated fade'><p><strong>Headline Added Successfully.</strong></p></div>";
            }
            elseif($_REQUEST['task_ok']==2){
                echo "<div id='message' class='updated fade'><p><strong>Headline Edited Successfully.</strong></p></div>";
            }
            elseif($_REQUEST['task_ok']==3){
                echo "<div id='message' class='updated fade'><p><strong>Headline Deleted Successfully.</strong></p></div>";
            }
            elseif($_REQUEST['task_ok']==4){
                echo "<div id='message' class='updated fade'><p><strong>Headlines Imported Successfully.</strong></p></div>";
            }
            elseif($_REQUEST['task_ok']==5){
                echo "<div id='message' class='updated fade'><p><strong>Headlines Edited Successfully.</strong></p></div>";
            }
            
        }
    }
    
    private function get_footer(){
        return "</div>";
    }
    
    function javascript_redirect($location) {
        // redirect after header here can't use wp_redirect($location);
        ?>
          <script type="text/javascript">
          <!--
          window.location= <?php echo "'" . $location . "'"; ?>;
          //-->
          </script>
        <?php
        exit;
    }

}