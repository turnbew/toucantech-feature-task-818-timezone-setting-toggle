TASK (HARD) DATE: 01.11.2017 - 16.11.17

TASK SHORT DESCRIPTION: 818 (Timezone setting toggle)

GITHUB REPOSITORY CODE: feature/task-818-timezone-setting-toggle

ORIGINAL WORK: https://github.com/BusinessBecause/network-site/tree/feature/task-818-timezone-setting-toggle


CHANGES
 
	IN FILES: 
	
		\network-site\system\codeigniter\language\english\db_lang.php
		
			ADDED LINE: $lang['db_value_field_table_exist'] = 'Value, field name or db table name exists.';
			
		
		\network-site\system\codeigniter\database\DB_query_builder.php
		
			ADDE CODE: 
			
				/**
				 * Determine if a value exist in table and coloumn
				 *
				 *	Created by Lajos Deli 17.11.2017
				 *	
				 *	@input
				 * 	- $value : string, required, this value will be checked
				 * 	- $field_name: string, rquired, coloumn's name
				 *	- $table_name: string, rquired, db table's name
				 *	- $additional_where: extra sql where part, not required, default: ''
				 *
				 * @return	bool
				 */
				public function value_exists($value, $field_name, $table_name, $addition_where = '')
				{
					if ($value === '' or $table_name === '' or $field_name === '')
					{
						return ($this->db_debug) ? $this->display_error('db_value_field_table_exist') : FALSE;
					}
					
					if ($addtional_where == '') {
						$query = $this->db->select('count (*) as c')
											->from($table_name)
											->where($field_name, $value)
											->get();
					} 
					else {
						$query = $this->db->select('count (*) as c')
											->from($table_name)
											->where($field_name, $value)
											->get($addition_where);
					}
					
					$result = $query->row_array();
					
					return ($result['c'] > 0) ? true : false;
				}
				
	
		\network-site\addons\default\modules\network_settings\language\english\network_settings_lang.php
		
			ADDED lines: 
				
				$lang['default_timezone'] = 'Default timezone';
				$lang['newsletters:send_on_timezone:label'] = 'Time zones';
				$lang['newsletters:instructions:send_on_timezone'] = 'Scheduled e-mail will be sent in determined time zone.';
	
	
		ADDED new file (with folder structure): \network-site\system\cms\modules\streams_core\field_types\timezone\field.timezone.php
	
			RELEVANT CODE IN IT: 
			
				/**
				 * Output a select input field which contains the timezones
				 *	
				 * @param	
				 *		- $data - array: form_slug, custom, value, max_length
				 *		- $entry_id - generally the id of the records, where the $data came 
				 * 		- $field - array: full of lot of stuffs
				 *
				 * @return	
				 *		- html formatted string (select input field with time zones)
				 *
				 */
				public function form_output($data, $entry_id, $field)
				{
					$this->load->helper('our_timezone'); 	// from addons\shared_addons\helpers\ folder
					$this->load->helper('our_form');		// from addons\shared_addons\helpers\ folder
					
					if ( ! $data['value'] and ! $entry_id)
					{
						$selected = (isset($field->field_data['default_value'])) ? $field->field_data['default_value'] : get_system_timezone();
					}
					else
					{
						$selected = $data['value'];
					}
					
					return form_chosen($data['form_slug'], get_timezone_options(), $selected, array('extra' => 'class="form-control"', 'required' => true));
					
				}
	
		\network-site\system\cms\core\MY_Controller.php
		
			ADDED CODE: $this->load->helper$this->load->helper(array('our_timezone')); //inside __construct function
			
	
		\network-site\addons\default\modules\network_settings\controllers\members.php
			
			//Inside index function
			
				CHANGED CODE:
					
					FROM: 
					
						foreach($users as $user) {
							if($user->person_type_names!='') {
								$user->person_type_names = explode(',', $user->person_type_names);
							}
						}
									
					TO: 
					
						foreach($users as $user) {
							$user->created = get_datetime_by_timezone($user->created);
							$user->last_login = get_datetime_by_timezone($user->last_login);
							if($user->person_type_names!='') {
								$user->person_type_names = explode(',', $user->person_type_names);
							}
						}
	
			//Inside function edit 
			
				FROM: 
				
					$donations_table = $this->load->view('fundraising/donations/member_profile.php', array('member' => $member, 'donations'=>$donations), true);
				
				TO: 
				
					$donations_table = $this->load->view('fundraising/donations/member_profile.php', array('member' => $member, 'donations'=>convert_datetime_by_timezone($donations)), true);
				
				******
			
				ADDED CODE: convert_datetime_by_timezone($member);

			//Inside function get_display_activity()		
				
				ADDED CODE: convert_datetime_by_timezone($activities);
					
				
			//Inside function consent_options()		
				
				ADDED CODE: convert_datetime_by_timezone($data);		
				
		\network-site\system\cms\config\config.php

			ADDED CODE: 
			
				define('default_time_format', 'd-m-Y H:i:s');
				define('default_system_timezone', 'Europe/London');
		
		\network-site\addons\default\modules\network_settings\controllers\payment_accounts.php
		
			CHANGED CODE: 
			
				FROM: 
				
					$this->data->accounts = $this->payment_accounts_m->get_all();
					
				TO:
				
					$this->data->accounts = convert_datetime_by_timezone($this->payment_accounts_m->get_all());
	
		\network-site\addons\default\modules\network_settings\views\general_settings\general_setups.php
		
			ADDED CODE: 
			
				<div class="row" style="margin-top: 20px;">
					<div class="form-group">
						<label for="default_timezone" class="col-sm-2 control-label"><?php echo lang('default_timezone'); ?></label>
						<div class="col-sm-5">
							<?=form_chosen('default_timezone',  $timezone_options, $default_timezone_selected, array('extra' => 'class="form-control"', 'required' => true))?>
						</div>
					</div>								
				</div>				
	
	
		\network-site\addons\default\modules\network_settings\views\content\events_form.php
	
			ADDED/CHANGED CODE:
			
				<!-- PREVIOUS timezone options: <?php //echo bbevent_timezone($post->time_zone != '' ? $post->time_zone : 'UTC'); ?>  -->
				<?=form_chosen('timezones',  $timezone_options, $default_timezone_selected, array('extra' => 'class="form-control"', 'required' => true))?>
	
	
* 		A bit help for me
*		print_r($galleries[0]); echo "<br><br>";
*		convert_datetime_by_timezone($galleries);
*		print_r($galleries[0]);		
*		//inside function view_galleries  ADDED CODE: convert_datetime_by_timezone($galleries);
			
		
		\network-site\addons\default\modules\history\controllers\history.php
		
			//inside function index  ADDED CODE: convert_datetime_by_timezone($recent_users);
		
		\network-site\addons\default\modules\bbusers\plugin.php
		
			//inside function most_recent_users  ADDED CODE: convert_datetime_by_timezone($most_recent);
		

		\network-site\addons\default\modules\messaging\controllers\messaging.php
		
			//inside function inbox  ADDED CODE: convert_datetime_by_timezone($messages);
			//inside function archived  ADDED CODE: convert_datetime_by_timezone($messages);
			//inside function deleted  ADDED CODE: convert_datetime_by_timezone($messages);
			//inside function read  ADDED CODE: convert_datetime_by_timezone($messages);
			//inside function sent  ADDED CODE: convert_datetime_by_timezone($messages);

		\network-site\addons\default\modules\messaging\models\spam_notifications.php

			CHANGED CODE: 
				FROM: public function get() TO: public function get($id = 0) //I just give an not use parameter to the function

		\network-site\addons\default\modules\connections\controllers\connections.php
		
			//inside function index  
				
				ADDED CODE: 
				
					convert_datetime_by_timezone($family_requests);
					.....
					$previous_connections = $this->connections_lib->group_by_week($previous_connections, 'actioned_on');
					convert_datetime_by_timezone($previous_connections);
		
					$outstanding_requests = $this->connections_lib->group_by_week($outstanding_requests, 'created_on');
					convert_datetime_by_timezone($outstanding_requests);


		\network-site\addons\default\modules\news\controllers\news.php
		
			//inside function _display_comments  ADDED CODE: convert_datetime_by_timezone($data);
			//inside function _news_sidebar  ADDED CODE: convert_datetime_by_timezone($data);
			//inside function _user_stories_sidebar  ADDED CODE: convert_datetime_by_timezone($data);
			//inside function annoucements  ADDED CODE: convert_datetime_by_timezone($this->data);
			//inside function author  ADDED CODE: convert_datetime_by_timezone($news);
			//inside function featuring  ADDED CODE: convert_datetime_by_timezone($news);
			//inside function index  ADDED CODE: convert_datetime_by_timezone($news);
			//inside function school  ADDED CODE: convert_datetime_by_timezone($news);
			//inside function view  ADDED CODE: convert_datetime_by_timezone($related_news);
												convert_datetime_by_timezone($comments_content);
			
			
			
		\network-site\addons\default\modules\bbevents\controllers\bbevents.php
		
			//inside function view  ADDED CODE: convert_datetime_by_timezone($event);
			//inside function index  ADDED CODE: 
										convert_datetime_by_timezone($events);
										convert_datetime_by_timezone($upcoming_events);
										convert_datetime_by_timezone($event_news);
			//inside function ajax_list  ADDED CODE: convert_datetime_by_timezone($events);
			//inside function event_log_in  ADDED CODE: convert_datetime_by_timezone($data->event);
			//inside function event_sign_up  ADDED CODE: convert_datetime_by_timezone($event);
			
		
		\network-site\addons\default\modules\bbevents\controllers\because_events.php
		
			//inside function ajax_list  ADDED CODE: convert_datetime_by_timezone($events);
			//inside function view  ADDED CODE: convert_datetime_by_timezone($event);
			
		
		\network-site\addons\default\modules\network_settings\controllers\fundraising.php
			
			//inside function donations  ADDED CODE: convert_datetime_by_timezone($payments, array('payment_date_received' => 'd-m-Y'));
			//inside function donationdetails  ADDED CODE: convert_datetime_by_timezone($payment);
			//inside function mysupport  ADDED CODE: convert_datetime_by_timezone($recent_donations);
	
		\network-site\addons\default\modules\network_settings\controllers\activity_tracker.php
			
			//inside function index  ADDED CODE:  convert_datetime_by_timezone($activities);
			
	
		\network-site\addons\default\modules\network_settings\controllers\documents.php
		
			//inside function index  ADDED CODE:  convert_datetime_by_timezone($documents['data']);
	
		\network-site\system\cms\libraries\Streams\drivers\Streams_ap.php
		
			//inside function entries_table  ADDED CODE:  convert_datetime_by_timezone($data['entries']);
	
		\network-site\addons\default\modules\network_settings\controllers\activity_tracker.php
			
			//inside function index  ADDED CODE:  convert_datetime_by_timezone($activities);
			
	
		\network-site\addons\default\modules\network_settings\controllers\email_center.php
			
			//inside function index  ADDED CODE:  convert_datetime_by_timezone($this->data);
			
	
		\network-site\addons\default\modules\network_settings\controllers\messages.php
			
			//inside function index  ADDED CODE:  convert_datetime_by_timezone($messages);
			//inside function view_thread  ADDED CODE:  convert_datetime_by_timezone($messages);
		
		\network-site\addons\default\modules\network_settings\controllers\shop\admin_orders.php
		
			//inside function index  ADDED CODE:  convert_datetime_by_timezone($this->data);
	
	
		\network-site\addons\default\modules\network_settings\controllers\content.php
		
			//inside function index  ADDED CODE:  convert_datetime_by_timezone($news);			
			//inside function announcements ADDED CODE: convert_datetime_by_timezone($news);
			//inside function events ADDED CODE: convert_datetime_by_timezone($events);		
			//inside function event_registrations ADDED CODE: convert_datetime_by_timezone($data);		
			//inside function edit_event
				
				ADDED CODE:  
				
					$this->load->model('general_setups_m');

					......
				
					$default_timezone_selected = ($event->time_zone != '') ? $event->time_zone : default_timezone;
					
					......
				
					->set('timezone_options', get_timezone_options()) //timezone_options form our_timezone_helper file
					->set('default_timezone_selected', $default_timezone_selected) 
							
	
		\network-site\addons\default\modules\network_settings\controllers\general_settings.php
		
			ADDED CODE 1: 
				
				//Inside function general_setups
				$this->load->library('Options');
				
				....
				
				'default_timezone' => array(
                    'field' => 'default_timezone',
                    'label' => 'Default timezone',
                    'rules' => 'required'
                )
				
				..... 
				
				'default_timezone' 					=> $this->input->post('default_timezone') 
				
				.....
				
				->set('timezone_options', get_timezone_options()) //from shared_addons/helper/our_timezone_helper 
				->set('default_timezone_selected', $this->general_setups_m->get_by_slug('default_timezone')->value)
	
	
			CHANGED CODE: //about at six placesű
			
				FROM: 
					
					$this->data['updated_on'] = $item->updated_on;
					
				TO:
				
					$this->data['updated_on'] = get_datetime_by_timezone($item->updated_on);
	
		ADDED NEW FILE: \network-site\addons\shared_addons\helpers\our_timezone_helper.php

			CODE IN IT: 
			
				/*
					Functions in this helper file:
				
					get_datetime_by_timezone(
											@param1: $time - can be "now" or any date format or timestamp, non required, default "now" 
											@param2: $timezone - name of timezone, required, 
											@param3: $just_timestamp - logical (true/false), non required, default false 
											@param4: $date_format - can be logical(true/false) or date format, non  required, default false
										) 
										@return -> DateTime object or timestamp or date format
					
					
					convert_datetime_by_timezone(
											@param1: &$object: can be array or std object, required
											@param2: $fields: array: list of date fields in array, not required
											@param3: $timezone - name of timezone, not required
										) 
										@return -> VOID (we'll get the result on input $object)
					

					get_timezone_regions(void) 
										@return -> array[regions of timezones] 
								
								
					get_timezones(void) 
										@return -> array[timezone's name => UTC offset]
										
										
					get_timezone_options(void) 
										@return -> array[timezone's name => city + am and pm format + UTC offset]
					
					
					get_timezone_dst_offset(
											@param1: $timezone - name of timezone, required, 
											@param2: $return - can be "offset" or "logical", non required, default "offset"
										)
										@return -> true/false (if $return = "logical") or 0/3600 (if $return = "offset")
				
					
					get_timezone_utc_offset(
											@param1: $timezone - name of timezone, required, 
										) 
										@return -> utc offset in seconds of thetimezone (offset is compared to GMT + 00:00)
								
			
					get_timezone_transformation_table(
											VOID
										) 
										@return -> array : conversions between old version and new version of timezones	
				*/



				//if (function_exists('get_datetime_by_timezone'))
				if (true)
				{
					/*
					* Returns back recent date and time of the timezone  
					*
					* @input : 
					*	- $time: can be "now", string fromatted date-time or integer, default "now"				
					*	- $timezone: name of the timezone, default is default_timezone constant value which based on database value (general_setups table -> default_timezone)							
					*	- $return_type: can be: object, integer, datetime, any kind of date format i.e.: "Y-m-d H", or '', default is ''			
					*	- $format: date-time format of the return value, relevant if return value is datetime format
					*		
					* @return
					*	- date object initialized by the timezone, timestamp, datetime or any kind of format date
					*/
					function get_datetime_by_timezone($time = "now", $timezone = default_timezone, $return_type = "", $format = default_time_format) 
					{	
						if (in_array((string)$time, array('0', 'NULL', '', '0000-00-00', '0000-00-00 00:00:00'))) return $time;

						//if  $time is an integer
						if(filter_var($time, FILTER_VALIDATE_INT) )
						{
							if ($return_type == "") $return_type = 'integer'; 
							$time = date(default_time_format, $time);	//default_time_format from config/config.php file				
						} 
						
						$date_object = new DateTime($time);
						$date_object->setTimezone(timezone_open($timezone));

						if ($return_type == "") {
							$return_type = 'datetime'; 
						}
						
						switch ($return_type) {
							case 'integer':
									return $date_object->getTimestamp() + $date_object->getOffset();
								break;
							case 'datetime': 
									return date($format, $date_object->getTimestamp() + $date_object->getOffset()); 
								break;
							case 'object' : 
									return  $date_object;
								break;		
							default :
									return date($return_type, $timestamp);
								break;
						}
					}

				} //END !function_exists('get_datetime_by_timezone')	
			

			
			
			

			
				if (!function_exists('convert_datetime_by_timezone'))
				{
					/* Recursive function
					*
					* Working on input $object parameter
					*
					* @input : 
					*	- $object: can be array or std object, required
					*	- $field_and_formats: array: list of date fields with time format, i.e. 'example_date' => 'Y-m-d'
					*	- $timezone: name of the timezone, default is default_timezone constant value which based on database value (general_setups table -> default_timezone)				
					*		
					* @return
					*	- return VOID (we'll get result on input &$object 
					*/
					function convert_datetime_by_timezone(&$object, $field_and_formats = array(), $timezone = default_timezone) 
					{			
						$l = count($field_and_formats);
						if ($l == 0 or ($l > 0 and $l < 20)) 
						{
							//20 is the length of $fields array below, if you put more or take away some field, this number must be changed 
							//define an array which contains the commonly used coloumn names of date fields in database (so far)
							$fields = array(
								'created' => default_time_format, 
								'created_on' => default_time_format, 
								'updated' => default_time_format, 
								'updated_on' => default_time_format, 
								'last_login' => default_time_format, 
								'start' => default_time_format, 
								'finish' => default_time_format, 
								'date' => default_time_format, 
								'last_edited' => default_time_format, 
								'created_at' => default_time_format, 
								'uploaded' => default_time_format, 
								'payment_date_received' => default_time_format, 
								'gift_acknowledgement_date' => default_time_format, 
								'hmrc_submitted_date' => default_time_format, 
								'hmrc_received_date' => default_time_format,
								'end_date_start' => default_time_format,
								'end_date_finish' => default_time_format,
								'date_start' => default_time_format,
								'date_finish' => default_time_format,
								'actioned_on' => default_time_format,
							);
							
							//redefine date format in $fields array by $field_and_formats value 
							foreach ($field_and_formats as $field => $format) $fields[$field] = $format;

							$field_and_formats = $fields;
						}

						foreach($object as $key => &$mix) 
						{
							if (is_array($mix) or is_object($mix)) 
							{	
								//Recursive call if $mix is an object or array
								convert_datetime_by_timezone($mix, $field_and_formats, $timezone);	
							}
							else if (array_key_exists((string)$key, $field_and_formats)) 
							{
								//$format = (isset($formats[$key])) ? $formats[$key] : default_time_format;
								(is_object($object))
										? $object->$key = get_datetime_by_timezone($mix, $timezone, "", $field_and_formats[$key])
										: $object[$key] = get_datetime_by_timezone($mix, $timezone, "", $field_and_formats[$key]); 
							}
						}		
					}

				} //END !function_exists('convert_datetime_by_timezone')	

				
				
				
				
				
				
				if (!function_exists('get_timezone_regions'))
				{
					/*
					* Returns back to timezone regions
					*
					* @input : VOID
					*
					* @return
					*	- array: timezone regions 
					*/
					function get_timezone_regions() 
					{
						return array(
							'Africa' => DateTimeZone::AFRICA,
							'America' => DateTimeZone::AMERICA,
							'Antarctica' => DateTimeZone::ANTARCTICA,
							'Arctic' => DateTimeZone::ARCTIC,
							'Asia' => DateTimeZone::ASIA,
							'Atlantic' => DateTimeZone::ATLANTIC,
							'Australia' => DateTimeZone::AUSTRALIA,
							'Europe' => DateTimeZone::EUROPE,
							'Indian' => DateTimeZone::INDIAN,
							'Pacific' => DateTimeZone::PACIFIC
						);
					}
				} //END !function_exists('get_timezone_regions')	

						


						
						
						
				if (!function_exists('get_timezones'))
				{
					/*
					* Returns back all of the timezones
					*
					* @input : VOID
					*
					* @return
					*	- array: 'name_of_timezones' => 'UTC_offset'
					*/
					function get_timezones() {
						
						//Get timezone regions
						$timezone_regions = get_timezone_regions();
						
						//Get all of timezones from regions
						$timezones = array();
						foreach ($timezone_regions as $name => $mask)
						{
							$zones = DateTimeZone::listIdentifiers($mask);
							foreach($zones as $timezone)
							{						
								$zone_date = get_datetime_by_timezone("now", $timezone, "object");
								$timezones[$timezone] = $zone_date->getOffset();
							}
						}			
					}

				} //END !function_exists('get_timezones')	


				
				
			
			
			

				if (!function_exists('get_timezone_options'))
				{
					/*
					* Returns back a record which contains all of the known timezones by regions
					*
					* @input : VOID
					*
					* @return
					*	- array: 'name_of_timezones' => parameters of timezone (name, recent time in zone, UTC offset)  
					*/
					function get_timezone_options() {
						
						//Get timezone regions
						$timezone_regions = get_timezone_regions();
						
						
						//Get all of timezones from regions
						$timezones = array();
						foreach ($timezone_regions as $name => $mask)
						{
							$zones = DateTimeZone::listIdentifiers($mask);
							foreach($zones as $timezone)
							{
								$zone_time = new DateTime(NULL, new DateTimeZone($timezone));						

								$zone_date = get_datetime_by_timezone("now", $timezone, "object");

								$utc_offset = $zone_date->getOffset();
								$utc_offset_hour = floor($utc_offset / 3600);
								$utc_offset_hour_abs = abs($utc_offset_hour);
								$utc_offset_rest_min = (($utc_offset - ($utc_offset_hour * 3600))/60);			
								$utc = " - UTC" . 
										(($utc_offset_hour < 0) ? "-" : "+") . 									//- or + timezone
										(($utc_offset_hour_abs >= 10) ? "" : "0") . $utc_offset_hour_abs .  	//display hours
										":" . 																	//hours and minutes seperator
										(($utc_offset_rest_min == 0) ? "00" : $utc_offset_rest_min);			//display minutes
							
								$am_pm = $zone_time->format('H') > 12 
												? ' ('. $zone_time->format('g:i a'). ')' 
												: '';
								
								$timezones[$name][$timezone] = substr($timezone, strlen($name) + 1) . ' - ' . $zone_time->format('H:i') . $am_pm . " " . $utc;
							}
						}			
						
						$timezone_options = array();
						foreach($timezones as $region => $list) {
							$timezone_options[$region] = $list;
						}
						return $timezone_options;
					}

				} //END !function_exists('get_timezone_options')	
			

			
			
			
			
				if (!function_exists('get_timezone_dst_offset'))
				{	
					/*
					* Checks we are in dst (daylight-savings-time) or not
					*
					* @input 
					*	- $timezone: name of timezone
					*	- @return: calues can be: logical, offset (default)
					*
					* @return
					*	- true or false if $return = "logical"
					*	- offset value in seconds if $return = "offset" (value 0 if we aren't in dst and 3600 if we are in dst)
					*
					*/
					function get_timezone_dst_offset($timezone, $return = "offset")
					{
						$date = get_datetime_by_timezone("now", $timezone, "object");
						
						$in_dst = intval(date_format($date, "I"));
						
						return ($in_dst) 
									? (($return == "offset") ? 3600 : false)
									: (($return == "offset") ? 0 : true);
					}
					
				} //END !function_exists('get_timezone_dst_offset')	



				
			

			
				if (!function_exists('get_timezone_utc_offset'))
				{	
					/*
					* Returns back with utc offset (in seconds) by timezone
					*
					* @input 
					*	- $timezone: name of timezone
					*
					* @return
					*	- array: 
					*			utc offset in seconds, 
					*/
					function get_timezone_utc_offset($timezone)
					{
						$date = get_datetime_by_timezone("now", $timezone, "object");
						
						return $date->getOffset();
					}
					
				} //END !function_exists('get_timezone_utc_offset')		
						







				if (!function_exists('get_timezone_transformation_table')) 
				{
					/*
					 * return back a timezone conversions table which help the convert to previous used zones to the new ones
					 *
					 *
					*/
					function get_timezone_transformation_table() 
					{
						return	array(
							'UM12'		=> 'Pacific/Fiji',
							'UM11'		=> 'Pacific/Midway',
							'UM10'		=> 'Pacific/Honolulu',
							'UM9'		=> 'America/Yakutat',
							'UM8'		=> 'America/Dawson',
							'UM7'		=> 'America/Denver',
							'UM6'		=> 'America/Belize',
							'UM5'		=> 'America/New_York',
							'UM45'		=> 'America/St_Johns',
							'UM4'		=> 'America/Anguilla',
							'UM3'		=> 'America/Araguaina',
							'UM2'		=> 'America/Noronha',
							'UM1'		=> 'America/Scoresbysund',
							'UTC'		=> 'Europe/London',
							'UP1'		=> 'Europe/Amsterdam',
							'UP2'		=> 'Europe/Athens',
							'UP3'		=> 'Europe/Moscow',
							'UP35'		=> 'Asia/Tehran',
							'UP4'		=> 'Asia/Yerevan',
							'UP45'		=> 'Asia/Kabul',
							'UP5'		=> 'Asia/Karachi',
							'UP55'		=> 'Asia/Kolkata',
							'UP575'		=> 'Asia/Kathmandu',
							'UP6'		=> 'Asia/Omsk',
							'UP65'		=> 'Asia/Yangon',
							'UP7'		=> 'Asia/Bangkok',
							'UP8'		=> 'Asia/Brunei',
							'UP875'		=> 'Australia/Eucla',
							'UP9'		=> 'Asia/Chita',
							'UP95'		=> 'Australia/Darwin',
							'UP10'		=> 'Australia/Brisbane',
							'UP105'		=> 'Australia/Broken_Hill',
							'UP11'		=> 'Antarctica/Casey',
							'UP12'		=> 'Asia/Anadyr',
							'UP13'		=> 'Pacific/Auckland',
							'UP14'		=> 'Pacific/Apia'
						);
					}							
				}	
			

			
						
		NOTE: \network-site\addons\default\modules\firesale\details.php - REMOVED everything from here to the network_settings\details.php file
		
		\network-site\addons\default\modules\network_settings\details.php
		
			ADDED CODE 1:
			
				//INSIDE upgrade function (and install as well)
				
				if (version_compare($old_version, '2.0.62', 'lt')) 
				{
					$this->load->helper(array('our_timezone'));
					
					$this->_install_general_setups_table($this->_install_general_setups_added_records());

					//Add currency_name field to the firesale_orders table if not exists
					$this->_insert_new_currency_name_field('firesale_orders', 'shipping');
					
					//Add currency_name field to the event_registrations table if not exists
					$this->_insert_new_currency_name_field('event_registrations', 'gateway_account');

					//Update default_fundraising_payments table
					$this->db->set('payment_currency', 'GBP')->where('payment_currency', NULL)->update($this->db->dbprefix('fundraising_payments'));
					$this->db->set('default_payment_currency', 'GBP')->where('default_payment_currency', NULL)->update($this->db->dbprefix('fundraising_payments'));			

					//change time_zone coloumn varchar size to (50)
					$this->db->query('ALTER TABLE ' . $this->db->dbprefix('events') . ' MODIFY COLUMN time_zone VARCHAR(50)');	

					//get helper transformation array form shared_addons/helpers/our_timezone_helper
					$ttt = get_timezone_transformation_table(); 

					//Update old timezone values to new one in two tables (events and because_events)
					$this->_update_timezones($this->db->dbprefix('events'), 'time_zone', $ttt);
					$this->_update_timezones($this->db->dbprefix('because_events'), 'time_zone', $ttt);

					//Add new field to the newsletters_newsletters table (send_on_timezone) and set the value to the system timezone, where send_on is not null
					if (!$this->db->field_exists('send_on_timezone', $this->db->dbprefix('newsletters_newsletters'))) 
					{
						$sql = 	"ALTER TABLE  " . $this->db->dbprefix('newsletters_newsletters') . 
								"	ADD COLUMN send_on_timezone VARCHAR(50) NULL " . 
								"	AFTER send_on";
						
						if (! $this->db->query($sql)) return false;
						
						$this->db->set('send_on_timezone', get_system_timezone())
								 ->where('send_on IS NOT NULL', NULL, false)
								 ->where('send_on !=', '0000-00-00 00:00:00')
								 ->update($this->db->dbprefix('newsletters_newsletters'));
					}
					
					//Add new newsletters fields to data_fields table
					if (!$this->db->value_exists('send_on_timezone', 'field_slug', $this->db->dbprefix('data_fields'), 'field_namespace = "newsletters"')) 
					{
						//Add new field to newsletters newsletters stream
						$fields = array(
							array(
							'name' 		=> 'lang:newsletters:send_on_timezone:label',
							'slug'		=> 'send_on_timezone',
							'namespace'	=> 'newsletters',
							'type'		=> 'timezone' 
							)
						);
						$this->streams->fields->add_fields($fields);
						$field_id = $this->db->insert_id();
					}
					else 
					{
						$query = $this->db->select('id')
									->where('field_slug', 'send_on_timezone')
									->where('field_namespace', 'newsletters')
									->get($this->db->dbprefix('data_fields'));
						$result = $query->result_array();
						$field_id = $result[0]['id'];
					}
					
					//Insert data into data_field_assignments table
					if (!$this->db->value_exists($field_id, 'field_id', $this->db->dbprefix('data_field_assignments'), 'stream_id = "6"')) 
					{
						//Set order in data_field_assignments table first
						$this->db->set('sort_order', 'sort_order+1', FALSE)
									->where('sort_order >', 20)
									->where('stream_id', 6)
									->update($this->db->dbprefix('data_field_assignments')); 
					
						$data = array(
									'sort_order' => 21,
									'stream_id' => 6,
									'field_id' => $field_id,
									'is_required' => true, 
									'is_unique' => true, 
									'instructions' => 'lang:newsletters:instructions:send_on_timezone'
								);
						$this->db->insert($this->db->dbprefix('data_field_assignments'), $data);
					}	
				}

				//Added functions 
				
				private function _install_general_setups_table($records = array()) 
				{
				
					$this->db->query(
						"CREATE TABLE IF NOT EXISTS `" . $this->db->dbprefix("general_setups") . "` ( " .
						" `id` INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY, " .
						"  `slug` varchar(100) NOT NULL,  " .
						"  `value` text, " .
						"  `updated_on` int(11) NOT NULL default 0, " .
						"  `last_updated_by` int(11) " .
						") ENGINE=InnoDB  DEFAULT CHARSET=utf8;"
					);	

					if (count($records) > 0) 
					{ 
						$table = $this->db->dbprefix('general_setups');
						foreach ($records as $key => $record) 
						{
							$num = $this->db
											->select('count(*) as num')
											->from($table)
											->where('slug', $record['slug'])
											->get()
											->row()
											->num;		
							if ($num == 0) {
								$this->db->insert($table, $record);
							}
						}
					}
				}
				
				
				
				private function _install_general_setups_added_records() 
				{
					$now = time(); 			
						
					return	array (	
								array (
									'slug' => 'default_currency',
									'value' => 'GBP', 
									'updated_on' => $now,
									'last_updated_by' => $this->current_user->id
								), 
								array (
									'slug' => 'default_currency_html_entity',
									'value' => '&#163;', 
									'updated_on' => $now,
									'last_updated_by' => $this->current_user->id
								) , 
								array (
									'slug' => 'default_timezone',
									'value' => 'Europe/London;', 
									'updated_on' => $now,
									'last_updated_by' => $this->current_user->id
								) 
							);
				}	

				
				
				
				private function _insert_new_currency_name_field($table_name, $after_coloumn) 
				{
					$table = $this->db->dbprefix($table_name);
					if (!$this->db->field_exists('currency_name', $table)) 
					{
						$sql =  "ALTER TABLE " . $table . " " . 
								"	ADD COLUMN currency_name VARCHAR(10) NULL DEFAULT 'GBP' " .
								"	AFTER " . $after_coloumn;
						
						return $this->db->query($sql);		
					}			
				}

				
				
				
				/*
				*
				* Update imezones in database tables
				*
				* @input: 
				*	$table : name of database table
				*	$timezone_field: name of table coloum which contains the zone names
				*	$ttt: array (tct = timezone_conversation_table) ... helper table, convertin previous used timezones to new ones
				*
				*/
				private function _update_timezones($table, $timezone_field = 'time_zone', $ttt = array())
				{	
					if (count($ttt) > 0) {
						foreach ($ttt as $old_zone => $new_zone) {
							$this->db
								->set(array(
										$timezone_field => $new_zone
									))
								->where($timezone_field, $old_zone)
								->update($table);
						}				
					}
				}	
	
					


				
		
