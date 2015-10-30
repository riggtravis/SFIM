© (C) Copyright 2015 Travis Rigg
© Copyright phpBB Limited 2003-2014
This project is distributed under the terms of the GNU General Public License v2

This file is part of SFIM.

    SFIM is free documentation: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 2 of the License, or
    (at your option) any later version.

    SFIM is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with SFIM.  If not, see <http://www.gnu.org/licenses/>.

# Textual Entity Relational Model
Based on searching around I have determined that probably the best forum to
target for allowing users to port their old forums into new forum management
systems is phpBB. This is not because phpBB is old and crufty and bad. This is
because phpBB is wide spread, and likely has a large range of utilities
available. SFIM should do everything in its power to be accessible to the
largest number of people, even if it isn't at first perceived as necessary.
The reason for this is not only immediate portability of data, but also
immediate portability of skills. Users, administrators, moderators, and even
programmers will still be able to use their body of knowledge of interacting
with forums to interact with a SFIM forum.

The phpBB platform is available under the terms of the GNU GPL 2.0. In the
spirit of this license, SFIM, the entity relational model will also be available
under these same terms. This is because SFIM's entity relational model is based
heavily on phpBB's, so in effect, SFIM is a stripped down fork of phpBB.

That said, programs that are written to take advantage of SFIM are not part of
SFIM, and can be released under their own licenses. For example, all of the
reference implementations that are part of the SFIM project will be available
under the MIT license.

## Tables
###		1.	acl_groups					-Permission roles assigned
Documentation of acl_groups has been deleted. Unfortunately, this means that I
will have to reverse engineer acl_groups by looking at how the code base of
pbpBB interacts with acl_groups.

I think that this table is actually a description of a relational entity. It
relates a user to many roles and roles to many users. However, some of the other
tables seem to indicate that this is no longer how roles are assigned and that
users are only assigned one role, meaning that this table has been deprecated
within the phpBB project. I think that this is a mistake because it is not a
very flexible situation that could possibly lead to a proliferation of roles and
user rights that is not a very clear and concise method of knowing which users
have what permissions.

####			Keys:
    group_id		unsigned int	Primary	Key
    group_name	varchar				Unique	Name of the group
    role_id			unsigned int	Foreign	Reference to acl_roles table
    user_id			unsigned int	Foreign	Reference to acl_people table

####			Attributes
    description	text					What is this group for?

###		2.	acl_options					-List of possible permissions
####			Keys:
    auth_option_id:	unsigned int	Primary	Key
    auth_option:		varchar				Unique	Description of the permission

####			Attributes:
    is_global			boolean	Is this permission given globally?
    is_local			boolean	Is this permission given on a by forum basis?
    founder_only	boolean	Is this permission given only to the community founder

###		3.	acl_roles						-Permission roles
Two unused attributes from this table were ignored.

####			Keys:
role_id	unsigned int	Primary	Key

####			Attributes:
    role_name					varchar	Title of the role
    role_description	text		What are the responsibilities of this role?

###		4.	acl_assignments			-Permissions each role contains
####			Keys:
    role_id					unsigned int	Foreign	References acl_roles table
    auth_option_id	unsigned int	Foreign	References acl_options table

role_id and auth_option_id together make the primary key for this table. I
suspect that they are both foreign keys as well, but I am not completely sure. I
believe that this makes this table a weak entity. Frankly, I'm going to see if I
can implement a different solution to this problem that is less idiosyncratic.

This is a relational entity. It allows there to be a many to many relationship
between roles and options. This means that a single role can be assigned many
responsibilities/permissions and that a single responsability/permission can be
held/performed by many roles.

####			Attributes:
    auth_setting	tinyint	Three possible settings, yes, no, or never.

All fields of this table must be set.

I believe that auth_setting has to do with authentification based on its name. I
think that what this has to do with is whether or not a user has to be logged in
or not in order to do a specified task.

###		5.	acl_people					-Permissions assigned
####			Keys:
    user_id					unsigned int	Foreign	References people table
    forum_id				unsigned int	Foreign	References forums table
    auth_option_id	unsigned int	Foreign	References acl_options

####			Attributes:
    auth_setting	tinyint	Three possible settings, yes, no, or never.

All attributes of this table must be set.

Similar to auth_setting in acl_roles_data, this is to do with authentification.
I'm not entirely sure what this means or why it is set twice in two different
places. Possibly I can clarify this through different naming conventions.

This table does not have a primary key or any gauruntee that an entry is unique.
This is not a good way of doing things, and should probably be modified in some
way.

###		6.	attachments					-Information on attachments
####			Keys:
    attach_id					unsigned int	Primary	Key		  							auto_increment
    post_message_id		unsigned int	Foreign References posts table
    topic_id					unsigned int	Foreign	References topics table
    poster_id					unsigned int	Foreign	References people table
    physical_filename	varchar				Unique	name of the file stored physically

post_msg_id, topic_id, and poster_id are all required fields.

####			Attributes:
    in_message			boolean				Is this attachment in a private message false
    is_orphan				boolean				Is this attachment unused?							 true
    real_filename		varchar				Name of the file as the user uploaded it.
    download_tally	unsigned int	How many times the file has been viewed?		0
    attach_comment	text					Comment
    extension				varchar				What kind of file is this? .png? .jpg?
    mimetype				varchar				How should the web browser handle this?
    file_size				unsigned int	How big is this file in bytes?							0
    filetime				timestamp			SQL timestamp, automatically generated.
    thumbnail				boolean				Is there a thumbnail for this file?					0

The file size should only allow sane file sizes. This is a forum, not an image
storage service like imgur or flickr. Thumbnails in the filesystem have the
prefix "thumb_"

The is_orphan field will be used for the cleansing of files that are wasting
space on the server. It should be possible for administrators to run a purge
directly or to have a scheduled cleaning utility.

Filetime is a required field and should be automatically created when an entry
to the table is created.

###		7.	bans							-Banned users, ip addresses, or emails
This table doesn't make much sense. I think the best way to do this would be to
separate this table into three different tables. the user_bans, the
ip_bans, and the email_bans. However, for v0.1 of SFIM I would like to keep as 
close as possible to the implementation used by phpBB so that the DNA of SFIM is
clearer. It must be kept in mind that at this early stage SFIM is a fork of
phpBB's underlying schema.

All of that said, fields in this banlist table that do not make sense will be
removed. An example of this is ban_reason and ban_give_reason. Having different
reasons than the reasons given to a user or a community is rude, and I would
prefer to ensure that people on a SFIM based forum have an enjoyable experience.

####			Keys:
    ban_id			unsigned int	Primary	Key
    user_id			unsigned int	Foreign	References people table
    ip_addr			varchar				Unique	Banned IP address
    email_addr	varchar				Unique	The email address of someone being banned

####			Attributes:
    ban_date			timestamp		When did the user get banned?
    ban_end_date	timestamp		When will the user be allowed to rejoin activities
    ban_exclude		boolean			Exclude this user from a general ban?
    ban_reason		text				Why was this user banned?

One of the following must be set for the entry to be valid:
* user_id
*	ip_address
*	email

###		8.	bbcodes							-Custom BBCodes
This table will likely not be a part of SFIM. It is currently a place holder in
case this changes before the first release, which will be v0.1. The reason that
it is likely to be abandoned is that it is not implementation agnostic, it
should be possible to use a lightweight markup language that is selected by the
implementers or administrators of a SFIM distribution forum. The other factor in
determining that this table doesn't have much place in the SFIM standard is that
phpBB really doesn't make much use of this table itself.

###		9.	bookmarks						-Bookmarked topics
####			Keys:
    topic_id	unsigned int	Foreign	References topics table
    user_id		unsigned int	Foreign	References people table

####			Attributes:
    order_ id	unsigned int	Keeps all bookmarked topics in order.

All keys and attributes of this table must be set.

I'm not clear on what this order_id does. Also this table does not have a
primary key. I believe that phpBB uses the combination of topic_id and user_id
as a primary key. It is important to note that topics and posts are not the same
things. A topic is a conversation. A post is a single message within a
conversation. A single message cannot exist without a topic.

###		10.	bots								-Spiders/Bots
####			Keys:
    bot_id	unsigned int Primary	Key	auto_increment

####			Attributes:
    bot_active	boolean				Is the bot in service?							true
    bot_name		varchar				What is the name of the bot?
    user_id			unsigned int	References people table
    agent				varchar				Undocumented
    ip_addr			varchar				What is the ip address of the bot?

user_id is a required field and cannot be left blank.

I have no idea what a spider is or what the agent attribute describes. This table
is likely to get modified for version 0.2.

###		11.	config							-Configuration information
The documentation for this table doesn't make sense to me. I think that I can
get away with not having this table at all. I think configuration can be handled
by a different method, such as flat files on the server.

###		12.	confirmations				-Contains session information for confirm pages
####			Keys:
    confirm			char	Primary	Key									auto_increment
    session_id	char	Foreign	References sessions

####			Attributes:
    confirm_type	int			The action the user was taking											0
    code					varchar	The character code that will be displayed.
    seed					int			The seed that should be used to initialize the RNG.	0
    attempts			int			The number of attempts that have been made.					0

###		13.	username_rules			-Disallowed usernames
####			Keys:
    disallow_id	unsigned int	Primary	Key

####			Attributes:
    disallow_username	varchar	What specifically is not allowed.

Implementers of SFIM interfaces are encouraged to make use of regular
expressions for this table.

###		14. drafts							-Drafts of future posts/private messages.
####			Keys:
    draft_id	unsigned int	Primary	Key														auto_increment
    user_id		unsigned int	Foreign	Reference to the people table.
    topic_id	unsigned int	Foreign	Reference to the topics table.
    forum_id	unsigned int	Foreign	Reference to the forums table.

####			Attributes:
    save_time			timestamp	What time was the message saved?
    draft_subject	varchar		potential title for the draft.
    draft_message	text			potential body for the draft.

###		15. extension_groups		-Extensions Groups, associate extensions with type
This table is used to help the forum system understand what to do with files.

####			Keys:
    group_id	unsigned int	Primary	Key	auto_increment

####			Attributes:
    group_name			varchar				What is the name of this group of associations
    cat_id					tinyint(2)		I think this has to do with categories			0
    allow_group			boolean				Is this group allowed to be posted?			false
    download_mode		boolean				Should this file be viewed immediately?	 true
    upload_icon			varchar				Icon to be shown when users are making uploads
    max_file_size		unsigned int	What's the biggest a file can be in bytes?	0
    allowed_forums	text					Where is this file type allowed?
    allow_in_pm			boolean				Is this sendable in private messages?  	false

I assume based on the allowed_forums field that there will be a way of
associating what file types are allowed in what forums. That said I think that
either way the allowed_forums field will get deleted because it's just not very
good practice.

###		16.	extensions					-Extensions (.xxx) allowed for attachments.
####			Keys:
    extenstion_id	unsigned int	Primary	Key
    group_id			unsigned int	Foreign	References the extension_groups

####			Attributes:
extension	varchar	What is the extension that you are trying to describe?

###		17.	forums							-Forums (Name, description, rules...)
####			Keys:
    forum_id			unsigned int	Primary Key												auto_increment
    parent_id			unsigned int	Foreign	References forums table. Parent of forum
    left_id				unsigned int	Foreign	References forums table. Part of b-tree
    right_id			unsigned int	Foreign	References forums table. Part of b-tree

An explanation of parent_id: parent_id indicates any forum that this forum is a
part of. An example of how this might be used is to create categories.

An explanation of left_id and right_id: The forum software creates a binary tree
to allow for fast searching of all forums. I don't think is very good for
clarity, so I might remove it in a future update. However, for now I leave it in
tact to increase clarity that SFIM is a fork of phpBB.

Despite the fact that I am leaving the binary tree methodology to increase the
clarity of genealogy of this product, I am not including any of the information
about the last post in this table. This should be decipherable through the posts
table based on time. The difference between these two sets of data is that using
a binary tree has a clear benefit, even if I will remove it and leave it up to
implementers of the SFIM standard to decide how to increase speed.

####			Attributes:
    parents						text					Holds a serialized array of parent forums.
    name							varchar				What is the name of the forum?
    description				text					What is this forum for?
    link							varchar				This attribute describes the URL of a forum
    password					varchar				This attribute does not make sense.
    style							tinyint				This attribute has 16 possible values			0
    image							varchar				What image represents this forum?
    rules							text					What are the expectations of users?
    rules_link				varchar				A web link to more thorough rules.
    topics_per_page		tinyint				This attribute has 16 possible values.		0
    type							tinyint				Is this a category, a post, or a link?		0
    status						tinyint(2)		FORUM_CAT, FORUM_Post, or FORUM_LINK			0
    post_total				unsigned int	How many posts in the forum?							0
    topic_total				unsigned int	How many valid discussions in the forum?	0
    topic_real_total	unsigned int	How many discussions, including invalid?  0
    forum_flags				tinyint				This attribute has 4 possible values.			0
    display_on_index	boolean				Will this forum appear on main page?   true
    enable_indexing		boolean				Allows for faster searching						 true
    enable_icons			boolean				I'm not sure what this does						 true
    enable_prune			boolean				Has to do with clearing poor posts		false
    prune_next				timestamp			What time will the next prune happen			0
    prune_days				tinyint				What days is the pruner scheduled to run	0
    prune_viewed			tinyint				This attribute has 16 possible values.		0
    prune_freq				tinyint				How often should prunes occur							0

For some of these attributes, I need to understand parsing text as phpBB handles
it. This means that some of the documentation in this table was written at
different times, which inherently impacts the ability of the maintainer to
remember what they were working on. So this section will also require revision
for version 0.2

Attributes related to text parsing are being left out of version 0.1. This is
because textual interpretation should remain implementation agnostic, and should
allow programmers to select the lightweight markdown language of their choice.

I would like to remove the style attribute from this database. Things having to
do with appearance should be handled by either the web interface or application.

I do not think that defining whether or not is entity is a category, a post, or
a link is truly necessary. I'm not sure enough yet to abandon it however until I
see it at work.

I don't see the necessity of listing both the number of topics and the number of
legitimate topics. In fact I don't see why we need either at all. This should be
handled by a query, not by storing a number.

The password field of this forum table isn't the best practice for having
private forums. A better method would be to use a method for having an invite
only forum. Examples of how to do this are to ban all IP address and then add
invited emails to the whitelist. Another way to do it is to perform a hashed
handshake for people attempting to use the forum. Either way, a list of people who
are allowed to use a forum is better than having a password for a forum.

###		18.	current_forum_users	-Store who is logged into protected forums
####			Keys:
    session_id	binary char		Foreign	References the sessions table.
    forum_id		unsigned int	Foreign	References the forums table.
    user_id			unsigned int	Foreign	References the people table.

####			Attributes:
This table does not have a primary key field but instead uses the combination of
session_id, forum_id, and user_id. Also, the user_id field does not need to be
stored in this table. The reason for this is that the session information drawn
from the sessions table will already contain the user information. Implementers
of SFIM who want to do things this way for speed purposes are encouraged to, but
this won't be part of future SFIM models because it is less clear.

###		19. forums_visits				-Unread post information is stored here.
####			Keys:
    user_id		unsigned int	Foreign	References the people table.
    forum_id	unsigned int	Foreign	References the forums table.

####			Attributes:
    mark_time	timestamp	Used for time based interactions

user_id and forum_id combine to create the primary key, and are therefor
required fields.

###		20.	watched_forums			-Subscribed forums
####			Keys:
    forum_id	unsigned int	Foreign	References the forums table.
    user_id		unsigned int	Foreign	References the people table.

####			Attributes:
    notify_status	boolean	Does the user need to be notified of recent changes?

This table does not have any primary key or a gauruntee that the information
held in it is unique. This will need to be remedied in the future.

###		21.	groups							-Usergroups
####			Keys:
    group_id	unsigned int	Primary	Key

####			Attributes:
    type						tinyint				16 possible values													1
    founder_manage	boolean				Is the founder responsible for this group?	0
    name						varchar				What is the name of this group?
    description			text					What is this group for and why does it exist?
    display					boolean				Can people see this group?							false
    avatar					varchar				What is the image for this group?
    avatar_type			tinyint(4)		What is the file type for the group avatar?	0
    avatar_width		tinyint(4)		How wide is the avatar image?								0
    avatar_height		tinyint(4)		How tall is the avatar image?								0
    rank						unsigned int	Undocumented																0
    color						varchar				Undocumented
    sig_char_num		unsigned int	How many characters in signatures allowed?	0
    receive_pm			boolean				Can users receive private messages?			false
    msg_limit_size	unsigned int	What's the biggest private message sendable	0
    legend					boolean				Does this group have a legend?					 true

I don't know what the color attribute is for, but I assume that it is for a
web interface setting. I still believe that settings such as this should be
separated out into a config file for the whole web interface to use. But since I
am unclear on what this is for, I will leave it for now an remove it for version
0.2 if it becomes clear that is merely an interface setting.

###		22. icons								-Post icons
I think that this table is purely about how the user interface will look. This
should be handled by an external configuration file.

###		23.	languages						-Installed languages
I believe that this table can be managed by external configuration files. On the
other hand, most forums I have interacted with have had a language menu that
acted very much as if it was interacting with a database. For this reason I am
leaving this table in tact with plans to research modifying it.

####			Keys:
    lang_id	tinyint(4)	Primary	Key

####			Attributes:
    iso						varchar	ISO code that identifies the language
    dir						varchar	Where the the language is stored
    english_name	varchar	What is the language called in English?
    local_name		varchar	What is the language called in itself?
    author				varchar	Undocumented

###		24.	logs								-Administration/Moderation/Error logs
####			Keys:
    log_id			unsigned int	Primary	Key											    auto_increment
    user_id			unsigned int	Foreign	References the people table
    forum_id		unsigned int	Foreign	References the forums table
    topic_id		unsigned int	Foreign	References the topics table
    reportee_id	unsigned int	Foreign	References the people table

####			Attributes:
    log_type  tinyint(2)    administrative, moderative, or error log?         0
    ip_addr		varchar(40)		IP address used by the user referenced by user_id
    time			timestamp			Time log was created.
    operation	text					A description of what happened
    data			text					Log info

###		25.	login_attempts			-Information about attempted logins
####			Keys:
    user_id	unsigned int	Foreign	References the people table

#### 			Attributes:
    ip							varchar		Where was the login attempt made from?
    browser					varchar		What kind of web browser was used?
    forwarded_for		varchar		I'm not sure what this means
    attempt_date		timestamp	When was the attempt made?											0
    username				varchar		What was the exact text the user typed?        	0
    username_clean	varchar 	What would this username be if sanitized?				0

This table needs a primary key.

###		26.	moderators					-Who is a moderator in which forum? (For display)
####			Keys:
    forum_id	unsigned int	Foreign	References the forums table
    user_id		unsigned int	Foreign	References the people table
    group_id	unsigned int	Foreign	References the groups table

####			Attributes:
    username					varchar	Username of a user
    group_name				varchar	Name of a group
    display_on_index	boolean	Should this information be stored on the index?	1

Username and group_name could probably both be gathered by querying user_id and
group_id respectively.

This table currently has no primary key or guaruntee of unique entries. This
will be modified in future updates.

###		27. modules							-Configuration of acp, mcp, and ucp modules
This table is extremely poorly documented, and does not have very understandable
attribute names. It's possible that it's not needed. It might also be super
important. From the phpBB documentation on modules, it's not clear what they are
used for. I believe that SFIM will be fine without it.

###		28.	poll_options				-Options text of all votes ("Yes", "No", ...)
Because there are so many web services that allow users to create polls, I think
it is best to plan on supporting these platforms in future updates that to have
this be part of the base design.

###		30.	posts               -Topics posts
####			Keys:
    post_id		unsigned int	Primary Key													auto_increment
    topic_id	unsigned int	Foreign	References the topics table
    forum_id	unsigned int	Foreign	References the forums table
    poster_id	unsigned int	Foreign	References the people table
    edit_user	unsigned int	Foreign	References the people table

####			Attributes:
    poster_ip_addr		varchar				The IP address where the post originated.
    post_date					timestamp			When was the post made?							  		0
    approved					boolean				Has the post been approved?         	 true
    reported					boolean				Has the post been reported?						false
    enable_lml				boolean				Does the user want to use markup?			 true
    enable_smilies		boolean				Does the user want to use smilies?		 true
    enable_magic_url	boolean				Does the user want automatic URLs?		 true
    enable_signature	boolean				Display signature?                     true
    username					varchar				Who posted this?
    subject						varchar				What are we talking about?
    text							text					What is being said?
    checksum					varchar				Verifies the post
    attachment				boolean				Are there attached files?							false
    postcount					boolean				Undocumented													 true
    edit_date					timestamp			When was this post edited?								0
    edit_tally				smallint(4)		How many times has the post been edited?	0
    edit_locked				boolean				Has the post been blocked from editing?		1

icon_id has been excluded because the icons table has been deprecated.

The following are required fields:
* topic_id
*	forum_id
*	poster_id

###		31.	private_messages					-Private messages text
####			Keys:
    message_id	unsigned int	Primary	Key                         auto_increment
    root_level	unsigned int	Foreign	The initial message. References privmsgs
    author_id		unsigned int	Foreign	References the people table.

icon_id has been excluded because the icons table has been deprecated.

####			Attributes:
    author_ip_addr		varchar				IP address where the message originated.
    message_date			timestamp			When was the message created?							0
    enable_lml				boolean				Is this message raw text or lml?			 true
    enable_smilies		boolean				Should emoticons be pictures?					 true
    enable_magic_url	boolean				Automatically convert URLs to links?	 true
    enable_sig				boolean				Attach signature?				  						 true
    subject						varchar				What is this message about?
    text							text					What is the message?
    edit_reason				varchar				Why was the message edited?
    attachment				boolean				Does this message have an attachment?	false
    edit_date					timestamp			When was the message edited?							0
    edit_tally				smallint(4)   How many edits have there been?           0
    to_addr						text					Colon separated list of recipients.

author_ip is a required field

edit_user has been deprecated because only the sender should be allowed to edit
a private message.

The to_addr field is interesting, and I think I sort of understand why it is.
If there was an associative entity and a table representing it, it would be
easier for a malicious user to monitor traffic within the system. This way it's
a little easier to prevent that from happening. However, I think it might be
best to make associative entity and find a way to protect it.

###		32.	message_folders			-Custom private messages folders (for each user)
See privmsgs_rules

###		33.	message_rules				-Message rules, e.g. "if * then move * into folders
This table is poorly documented. For this reason, folders will not be a part of
base implementation in v0.1. They will be a high priority addition in the
future. This means that the privmsgs_folders table will also not be a required
table for v0.1.

###		34.	sent_messages				-Information (sender, new...) on private messages
####			Keys:
    message_id		unsigned int	Foreign	References the private_messages table
    recipient_id	unsigned int	Foreign	Who is the recepient?
    author_id			unsigned int	Foreign	Who is the sender?

####			Attributes:
    deleted		boolean	Has the user deleted this message?		false
    new				boolean	Is this message new?									true
    unread		boolean	Has the user read this message?				true
    replied		boolean	Has the user replied to this message?	false
    marked		boolean	Has the user marked this message?			false
    forwarded	boolean	Has the user sent this message on?		false

folder_id: see privmsgs_rules.

###		35.	profile_fields		-Custom profile fields (name, number characters...)
This table is poorly documented. It will be left out of v0.1, and possibly SFIM
entirely. All profile fields tables have been abandoned. This includes
the profile_lang table.

###		36.	ranks							-Ranks (Name, image, minimal # of posts)
It is not at all clear what this table does. It has been left out of the first
version of the SFIM standard.

###		37.	reports						-Reported posts
####			Keys:
    report_id	unsigned int	Primary	Key														auto_increment
    reason_id	unsigned int	Foreign References the reports_reasons table
    post_id		unsigned int	Foreign	References the posts table
    user_id		unsigned int	Foreign References the people table

####			Attributes:
    user_notify	boolean				Has the user been notified?					false
    closed			boolean				Has the report been closed?					false
    report_date	timestamp			When was the report filed?					0
    text				text					What text does the report display?

The following fields are required:
* reason_id
* post_id
* user_id

###		38. report_reasons		-Reasons for reported posts and disapprovals
####			Keys:
    reason_id	unsigned int	Primary	Key

####			Attributes:
    title				varchar		What is a succint title for this reason?
    description	text			What is a boilerplate description of this reason?
    order				smallint	16 possible values

###		39.	search_results		-Last searches
This series of these tables don't make sense to me. Searches should not be
stored in the database.

###		40.	sessions					-Sessions to identify people browsing the forum
####			Keys:
    session_id	unsigned int	Primary	Key
    user_id			unsigned int	Foreign	References to the people table

####			Attributes:
    last_visit_date	timestamp			When was the last time the user visited?		0
    start						unsigned int	When did the user log in.										0
    ip							varchar				Where is the user?
    browser					varchar				What kind of browser is the user using?
    page						varchar				What page is the user currently viewing?
    viewonline			boolean				I have no idea what this field does			 true
    autologin				boolean				Does the user want to be remembered?		false
    admin						boolean				Is this user an administrator?					false

user_id is a required field.

I have changed session_id to an unsigned int and given it the auto_increment
directive. I have done this because primary keys should be gaurunteed to be
unique.

I'm not sure the time field makes sense. That should probably be something
that's calculated through logic. In fact, I'm removing that field now.

I'm not sure, but I think that the page attribute should be a foreign key... But
I can't think of how that would be done since all of the different possible
pages are part of different tables. The two options are to leave this as is or
to find a way to modify the schema such that posts, topics, forums, and private
messages are all the same thing. I think this is probably an untenable task.
More than likely how this information will be stored is as a page determined by
the URI passed to the logic controller.

###		41.	session_keys			-Autologin feature
####			Keys:
    key_id	char					Primary	Key													SHA-2 hash
    user_id	unsigned int	Foreign	References the people table.

####			Attributes:
    last_ip					varchar				Where was the last place the user logged in?
    last_login_date	timestamp			When was the last time the user logged in?	0

user_id is a required field.

phpBB uses an MD5 hash, which is not secure. Also, because of collisions, the
method was using both key_id and user_id as a primary key. SHA-2 is known for
preventing collisions, so this should no longer be needed.

###		42.	trusted_sites			-Trusted websites for downloads
####			Keys:
    site_id	unsigned int	Primary	Key	auto_increment

####			Attributes:
    ip					varchar	Where is the website?
    hostname		varchar	What is the URL of the site's homepage?
    ip_exclude	boolean	Is this website untrusted?							false

###		43.	smilies						-Smilies (text => image)
This table doesn't make sense to me. As much as I would like to include smilies
for v0.1, I don't think it's feasible based on this table. It will be a high
priority feature for future updates.

###		44.	styles						-Style = template + theme + imageset
These style tables don't make any sense to me. I will be excluding them from the
SFIM standard.

###		45.	topics						-Topics in forums
####			Keys:
    topic_id			unsigned int	Primary	Key												auto_increment
    forum_id			unsigned int	Foreign	References the forums table.
    poster				unsigned int	Foreign	References the people table.
    first_post_id	unsigned int	Foreign	References the posts table.

icon_id has been excluded as the icons table has been deprecated.

####			Attributes:
    attachment				boolean				Do any posts have attachments?false
    pen_approval			boolean				Is this topic awaiting approval?			 true
    reported					boolean				Has this topic been reports?					false
    title							varchar				What is the title of this topic?
    creation_date			timestamp			When was the topic posted?							  0
    unstick_date			date					When will this topic stop being sticky?
    view_tally				unsigned int	How many views have happend?							0
    reply_total				unsigned int	How many replies have been approved?			0
    reply_real_total	unsigned int	How many replies of any kind are there?		0
    status						tinyint				UNLOCKED, LOCKED, MOVED										0
    type							tinyint				NORMAL, STICKY, ANNOUNCE, GLOBAL					0

The following are required fields:
* forum_id
* poster
* first_post_id

###		46.	topic_postings		-Who posted in which topic (used for the small dots)
####			Keys:
		user_id		unsigned int	Foreign References the people table.
		topic_id	unsigned int	Foreign References the topics table.

####			Attributes:
		topic_posted	boolean	Undocumented	false

Because this is an associative entity, in order to gauruntee unique entities,
user_id and topic_id together make up the primary key.

The attribute, topic_posted does not make sense to me.

###		47.	tracked_topics		-Unread post information is stored here
####			Keys:
		user_id 	unsigned int	Foreign References the people table.
		topic_id	unsigned int	Foreign	References the topics table.

####			Attributes:
		mark_time	timestamp	Last visit to this topic	0

Because this is an associative entity, in order to quaruntee unique entities,
user_id and topic_id together make up the primary key. The forum_id key has been
removed as this information will be stored in the topic entity.

###		48.	watched_topics		-Who wants notifications about changes to what topic
####			Keys:
		topic_id	unsigned int	Foreign References the topics table.
		user_id		unsigned int	Foreign	References the people table.

####			Attributes:
		notify_status	boolean	Does the user need to be notified of new posts?	false

Because this is an associative entity, in order to quaruntee unique entities,
user_id and topic_id together make up the primary key.

###		49.	memberships				-Groups of people.
This table describes the membership of people in groups. This is a poor table
name as it is not clear what it is that it represents. This will be renamed
before v0.1

####			Keys:
		group_id	unsigned int	Foreign References the groups table.
		user_id		unsigned int	Foreign	References the people table.

####			Attributes:
		group_leader	boolean	Is this user a leader of this group?
		user_pending	boolean	Is this user awaiting approval?

As this is an associative entity, group_id and user_id together make the
primary key in order to gauruntee uniqueness.

###		50.	people							-Registered people.
####			Keys:
		user_id					unsigned int	Primary	Key											auto_increment
		username				varchar				Unique	How the user is known in the community
		email						varchar				Unique	An email to contact the user with.

####			Attributes:
		user_type					tinyint				normal, inactive, bot, founder.						1
		permissions				text					A cached copy of the computed permissions.
		ip_addr						varchar				The IP address the user registered from.
		registration_date	date					When did the user sign up?
		password					varchar				SHA-2 hashed value of user password + salt.
		passchang_date		date					When did the user set their password?			0
		birthday					varchar				dd-mm-yyyy
		last_visit_date		timestamp			When was the last time the user visited?	0
		lastmark					timestamp			Last time the user marked forums read			0
		lastpost_time			timestamp			The time of the latest post by the user		0
		lastpage					varchar				The URL of the last page visited by the user
		last_confirm_key	varchar				Code used for security reasons
		last_search_time	timestamp			The last time user performed a search			0
		warnings					tinyint				The number of warnings the user has.
		last_warning			timestamp			The last time the user was warned.				0
		login_attempts		tinyint				Failed login attempts. Resets on success	0
		inactive_reason		tinyint(4)		The reason the user is inactive.					0
		inactive_date			timestamp			When the user's account became inactive		0
		posts							unsigned int	Amount of posts the user has posts				0
		language					varchar				The user's selected language
		timezone					decimal				The user's offest from UTC						 0.00
		dst								boolean				Is the user on Daylight Savings Time	false
		dateformat				varchar				The user's desired date format.	%d %m	%y %2
		new_message				tinyint				The number of new private messages.
		unread_message		tinyint				The number of unread private messages.
		last_message			date					The last time the user sent a message			0
		message_rules			boolean				Flag indicating if the user has rules	false
		full_folder				int						What to do when a folder is full				 -3
		emailtime					timestamp			The time the user sent an email						0
		topic_show_days		unsigned int	The maximum age of a topic to be shown		0
		topic_sortby_type	tinyint				author, replies, time, subject, views.		2
		topic_sortby_asc	boolean				Sort ascending or descending.					false
		post_show_days		unsigned int	Preferences for reading										0
		post_sortby_type	tinyint				author, subjuct, time											2
		post_sortby_asc		boolean				Sort ascending or descending.					 true
		notify						boolean				Notify of replies by deafault?						1
		notify_pm					boolean				Should the user be notified of new PMs?		1
		notify_type				tinyint				email, IM, both														0
		allow_pm					boolean				Does the user want to receive PMs?		 true
		allow_viewonline	boolean				Is the user visible or hidden?						1
		allow_viewemail		boolean				Does the user want to receive email?			1
		allow_massemail		boolean				Does the user want to receive mass email?	1
		avatar						varchar				Avatar's file name
		avatar_width			unsigned int	Width of the avatar.											0
		avatar_height			unsigned int	Height of the avatar.											0
		signature					text					The user's signature.
		from							varchar				Where is the user from?
		aim								varchar				What is the user's AIM nick?
		msn								varchar				what is the user's MSN nick?
		jabber						varchar				Jabber info
		website						varchar				URL of the user's website
		occupation				text					What does the user do?
		interests					text					What does the user like?
		activation_key		varchar				The key required to activate the account
		new_passowrd			varchar				The SHA-2 hash value of a random password

group_id does not make sense in this table because it is a many to many
relationship. It will be excluded.

I'm not convinced that username_clean is a necessity. It will be deprecated. The
reason I am currently thinking that it is that the logic controllers programmed
should be able to lower-case the username field through some function.

I actually like having a cached copy of permissions. This should make the system
faster in theory when a user just wants to review their permissions. They won't
have to query the database. They can just look at this field.

perm_form doesn't make any sense as a field. It is being left out.

Because the email field is being stored as a unique key, the email hash field to
check if it is a unique email address is no longer needed. The reason I am doing
it this way is that it is clearer, and already pretty darned fast. The problem
with storing the hash of the email address to make sure that the email is unique
is that the email address essentially has to be stored already in order for the
hash to even be useful.

last_search_time and emailtime is used by the system to keep people from flooding
the server with searches.

For formatting the date. For more information, please see
[this website:](http://www.cyberciti.biz/faq/linux-unix-formatting-dates-for-display/ "Unix Time Formatting")

The styling storage in the database design has been deprecated. For this reason
the style attribute of this table is no longer useful.

The color field has been left out. I think that should be handled by an external
configuration file.

The user_options field has been deprecated because it makes no sense, even when
it is documented.

avatar_type is being left out of this release. Avatars will be assumed to stored
in the filesystem for this release.

The signature bbcode fields have been deprecated because that should be handled
by the logic controllers, as discussed in other locations in this document.

regdate should be automatically created upon the creation of this table, and
will be a required field.

I'm not sure that the last private message time needs to be stored as this
information should be stored in the private messages table. However, this table
should also be protected from malicious people learning things that they should
not. For this reason I am leaving that field for now.

I'm much more positive that the last post time field should be removed. However,
this branch is only for fixing style and data storage types. I will create an
issue on github describing the problem of having unneeded things stored in silly
tables.

The following are required fields:
* user_birthday
* user_email_hash
* user_type

###		51.	warnings					-Warnings given to people
####			Keys:
		warning_id	unsigned int	Primary Key
		user_id			unsigned int	Foreign	References the people table.
		post_id			unsigned int	Foreign	References the posts table.
		log_id			unsigned int	Foreign References the logs table.

####			Attributes:
		warning_date	timestamp	When was the warning given?

###		52.	words							-Censored words
####			Keys:
		word_id	unsigned int	Primary	Key

####			Attributes:
		word				varchar	What is the banned text?
		replacement	varchar	What should the banned text be replaced with?

Users are encouraged to use regular expressions to implement word bans.

###		53. zebras						-Friends or foes
This table doesn't even begin to make sense.
