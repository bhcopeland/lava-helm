# Changelog

All notable changes to the LAVA Helm chart will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Initial upstream release of LAVA Helm chart
- Support for multiple deployment sizes (default, large)
- Autoscaling configuration for production workloads
- Backup and cleanup automation
- Comprehensive documentation and examples
- Security context configuration
- Service account automation
- Ingress configuration support

### Changed
- Updated default LAVA version to 2025.06
- Improved Chart.yaml with proper upstream metadata
- Enhanced values.yaml with comprehensive configuration options

### Fixed
- Resource allocation for different deployment sizes
- Storage persistence configuration
- Node selector compatibility

## [1.0.0] - 2025-11-18

### Added
- Initial release of LAVA Helm chart extracted from Linaro deployments
- Basic LAVA server, publisher, scheduler, and worker components
- ConfigMaps and Secrets management
- Persistent Volume Claims for data storage
- Horizontal Pod Autoscaler support
- Cronjobs for maintenance and cleanup
- Service and Ingress configuration
- External Secrets integration
- Job management for migrations and maintenance

### Features
- Multi-architecture support (x86_64, aarch64)
- Configurable resource sizing
- Environment-specific configurations
- Backup automation to S3-compatible storage
- Log compression and cleanup
- Health checks and monitoring
- Integration with Anubis proxy service
- CloudFront support for global distribution

### Documentation
- Comprehensive README with installation instructions
- Example configurations for different deployment scenarios
- Production-ready configuration examples
- Multi-instance deployment guidance

### Security
- Service account configuration
- Pod security context settings
- Network policies support (when enabled)
- Secret management integration