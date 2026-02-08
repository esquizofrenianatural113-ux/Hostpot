# MIKHMON - Hotspot Management System

![MIKHMON Logo](assets/img/logo.png)

MIKHMON is a modern, rebranded version of mikhmon - a comprehensive hotspot management system for MikroTik routers.

## Features

- 🔐 **Secure Authentication** - Secure login system with encrypted credentials
- 📊 **Dashboard** - Real-time monitoring of router resources and hotspot status
- 👥 **User Management** - Create, edit, and manage hotspot users and profiles
- 🎫 **Voucher Generation** - Generate and print vouchers for hotspot access
- 📈 **Reports** - Detailed reporting on usage and revenue
- 🎨 **Multiple Themes** - Light, Dark, Blue, Green, and Pink themes
- 📱 **Mobile Responsive** - Works on desktop and mobile devices
- 🔔 **Expire Monitor** - Automatic user expiration monitoring

## Requirements

- PHP 7.4 or higher
- Apache web server with mod_rewrite
- MikroTik RouterOS 6.x or 7.x
- PHP Extensions: pdo, zip

## Installation

### Manual Installation

1. Upload all files to your web server
2. Ensure the `config` directory is writable
3. Access the application via your browser
4. Default admin credentials:
   - Username: `mikhmon`
   - Password: `admin`

### Docker Installation

```bash
# Build and start containers
docker-compose up -d

# Access the application
http://localhost:8080
```

### Manual Docker Run

```bash
docker build -t mikhmon .
docker run -p 8080:80 -v $(pwd):/var/www/html mikhmon
```

## Configuration

### Router Connection

1. Log in to SKYNITY
2. Go to Settings > Router
3. Add your MikroTik router with:
   - Router IP/Hostname
   - Username
   - Password
   - Hotspot Name
   - Currency

### Theme Selection

SKYNITY supports multiple themes:
- Light (default)
- Dark
- Blue
- Green
- Pink

Change themes from the sidebar menu.

## API Endpoints

### GET Endpoints
- `?session/get_sys_resource` - System resource information
- `?session/get_hotspot_active` - Active hotspot users
- `?session/get_hosts` - Hotspot hosts
- `?session/get_interface` - Network interfaces
- `?session/get_profile` - User profiles
- `?session/get_users` - All users
- `?session/get_report` - Usage reports

### POST Endpoints
- `?session/add_user` - Add new user
- `?session/update_user` - Update user
- `?session/add_userprofile` - Add user profile
- `?session/generate_voucher` - Generate vouchers
- `?session/logout` - Logout

## File Structure

```
mikhmon/
├── assets/
│   ├── css/          # Stylesheets
│   ├── fonts/        # Font Awesome icons
│   ├── img/          # Images and logos
│   └── js/           # JavaScript files
├── config/           # Configuration files
├── core/             # Core application files
├── get/              # GET request handlers
├── post/             # POST request handlers
├── template/         # Print templates
├── view/             # UI views
├── nginx.conf        # Nginx configuration
├── Dockerfile        # Docker build file
└── docker-compose.yml # Docker compose file
```

## Security

- All passwords are encrypted using custom encoding
- Session-based authentication
- Protected against XSS attacks
- CSRF protection enabled

## Troubleshooting

### Connection Issues

1. Check if MikroTik API is enabled (System > API)
2. Verify router IP and credentials
3. Ensure port 8728 is accessible

### Login Problems

1. Clear browser cache and cookies
2. Check session configuration in config.php
3. Verify file permissions (config directory must be writable)

## Contributing

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## License

This project is licensed under the MIT License.

## Credits

- Original mikhmon by Laksamadi Guko
- MIKHMON - Modern Rebrand
- Font Awesome for icons
- Highcharts for charts
- jQuery for DOM manipulation

## Support

For support, please open an issue on GitHub or contact the development team.

---

**MIKHMON** - Modern Hotspot Management System
