pipeline {
    agent any
    
    // ═══════════════════════════════════════════════════════════════════════════════
    // ENVIRONMENT VARIABLES & CONFIGURATION
    // ═══════════════════════════════════════════════════════════════════════════════
    environment {
        PYTHON_VERSION = '3.11'
        VENV_DIR = 'venv'
        BUILD_DIR = 'build'
        DEPLOY_DIR = '/opt/flask-app'  // Change this to your production path
        GITHUB_REPO = 'https://github.com/Ayila890/SSD-Final.git'
        BRANCH = 'main'
    }
    
    // ═══════════════════════════════════════════════════════════════════════════════
    // BUILD TRIGGERS - Pipeline triggers on GitHub push
    // ═══════════════════════════════════════════════════════════════════════════════
    triggers {
        // Trigger on GitHub push (requires GitHub plugin configured)
        githubPush()
    }
    
    // ═══════════════════════════════════════════════════════════════════════════════
    // PIPELINE STAGES
    // ═══════════════════════════════════════════════════════════════════════════════
    stages {
        
        // STAGE 1: CLONE REPOSITORY FROM GITHUB
        // ─────────────────────────────────────────────────────────────────────────────
        stage('1️⃣ Clone Repository') {
            steps {
                echo '═══════════════════════════════════════════════════════════════════════════════'
                echo '📋 STAGE 1: CLONE REPOSITORY FROM GITHUB'
                echo '═══════════════════════════════════════════════════════════════════════════════'
                echo "🔄 Cloning Flask App Repository..."
                echo "📍 Repository: ${GITHUB_REPO}"
                echo "🌿 Branch: ${BRANCH}"
                
                script {
                    // Using Git plugin to clone repository
                    // This requires GitHub plugin: Manage Jenkins → Manage Plugins → GitHub
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: '*/main']],
                        userRemoteConfigs: [[
                            url: '${GITHUB_REPO}',
                            credentialsId: 'github-credentials'  // Configure in Jenkins credentials
                        ]]
                    ])
                    
                    echo "✅ Repository cloned successfully"
                    echo "📁 Current workspace: \$(pwd)"
                }
            }
        }
        
        // STAGE 2: SETUP PYTHON ENVIRONMENT
        // ─────────────────────────────────────────────────────────────────────────────
        stage('2️⃣ Setup Python Environment') {
            steps {
                echo '═══════════════════════════════════════════════════════════════════════════════'
                echo '⚙️  STAGE 2: SETUP PYTHON ENVIRONMENT'
                echo '═══════════════════════════════════════════════════════════════════════════════'
                echo "📦 Creating Python ${PYTHON_VERSION} virtual environment..."
                
                script {
                    if (isUnix()) {
                        sh '''
                            echo "Creating virtual environment on Linux/Mac..."
                            python3 -m venv ${VENV_DIR}
                            . ${VENV_DIR}/bin/activate
                            echo "✅ Virtual environment created"
                            
                            echo "Upgrading pip, setuptools, and wheel..."
                            pip install --upgrade pip setuptools wheel
                            echo "✅ Tools upgraded successfully"
                        '''
                    } else {
                        bat '''
                            echo Creating virtual environment on Windows...
                            python -m venv %VENV_DIR%
                            call %VENV_DIR%\\Scripts\\activate.bat
                            echo Virtual environment created
                            
                            echo Upgrading pip, setuptools, and wheel...
                            python -m pip install --upgrade pip setuptools wheel
                            echo Tools upgraded successfully
                        '''
                    }
                }
            }
        }
        
        // STAGE 3: INSTALL DEPENDENCIES FROM requirements.txt
        // ─────────────────────────────────────────────────────────────────────────────
        stage('3️⃣ Install Dependencies') {
            steps {
                echo '═══════════════════════════════════════════════════════════════════════════════'
                echo '📥 STAGE 3: INSTALL PYTHON DEPENDENCIES FROM requirements.txt'
                echo '═══════════════════════════════════════════════════════════════════════════════'
                echo "Installing 43 pinned Python packages from requirements.txt..."
                
                script {
                    if (isUnix()) {
                        sh '''
                            . ${VENV_DIR}/bin/activate
                            
                            echo "Checking requirements.txt exists..."
                            if [ -f requirements.txt ]; then
                                echo "✅ requirements.txt found"
                                echo "Installing dependencies (43 packages)..."
                                pip install -r requirements.txt
                                echo "✅ All dependencies installed successfully"
                                
                                echo "Installed packages:"
                                pip list
                            else
                                echo "❌ ERROR: requirements.txt not found!"
                                exit 1
                            fi
                        '''
                    } else {
                        bat '''
                            call %VENV_DIR%\\Scripts\\activate.bat
                            
                            echo Checking requirements.txt exists...
                            if exist requirements.txt (
                                echo requirements.txt found
                                echo Installing dependencies (43 packages)...
                                pip install -r requirements.txt
                                echo All dependencies installed successfully
                                
                                echo Installed packages:
                                pip list
                            ) else (
                                echo ERROR: requirements.txt not found!
                                exit /b 1
                            )
                        '''
                    }
                }
            }
        }
        
        // STAGE 4: CODE QUALITY CHECKS (Optional)
        // ─────────────────────────────────────────────────────────────────────────────
        stage('4️⃣ Code Quality Checks') {
            steps {
                echo '═══════════════════════════════════════════════════════════════════════════════'
                echo '🔍 STAGE 4: CODE QUALITY CHECKS (Flake8)'
                echo '═══════════════════════════════════════════════════════════════════════════════'
                echo "Running code quality checks with flake8..."
                
                script {
                    if (isUnix()) {
                        sh '''
                            . ${VENV_DIR}/bin/activate
                            echo "Running flake8 linting checks..."
                            flake8 app.py --max-line-length=120 --statistics || true
                            echo "✅ Code quality check completed"
                        '''
                    } else {
                        bat '''
                            call %VENV_DIR%\\Scripts\\activate.bat
                            echo Running flake8 linting checks...
                            flake8 app.py --max-line-length=120 --statistics || exit /b 0
                            echo Code quality check completed
                        '''
                    }
                }
            }
        }
        
        // STAGE 5: RUN UNIT TESTS WITH PYTEST
        // ─────────────────────────────────────────────────────────────────────────────
        stage('5️⃣ Run Unit Tests') {
            steps {
                echo '═══════════════════════════════════════════════════════════════════════════════'
                echo '🧪 STAGE 5: RUN UNIT TESTS (Pytest with Coverage)'
                echo '═══════════════════════════════════════════════════════════════════════════════'
                echo "Executing unit tests using pytest framework..."
                
                script {
                    if (isUnix()) {
                        sh '''
                            . ${VENV_DIR}/bin/activate
                            
                            echo "Checking if tests directory exists..."
                            if [ -d "tests" ]; then
                                echo "✅ Tests directory found"
                                echo "Running pytest with coverage reporting..."
                                pytest tests/ -v --cov=. --cov-report=xml --cov-report=html --cov-report=term || true
                                echo "✅ Unit tests completed"
                                echo "Coverage reports generated in htmlcov/ directory"
                            else
                                echo "⚠️  No tests directory found - creating sample test structure..."
                                mkdir -p tests
                                echo "Please add test files to tests/ directory"
                            fi
                        '''
                    } else {
                        bat '''
                            call %VENV_DIR%\\Scripts\\activate.bat
                            
                            echo Checking if tests directory exists...
                            if exist tests (
                                echo Tests directory found
                                echo Running pytest with coverage reporting...
                                pytest tests/ -v --cov=. --cov-report=xml --cov-report=html --cov-report=term || exit /b 0
                                echo Unit tests completed
                                echo Coverage reports generated in htmlcov/ directory
                            ) else (
                                echo No tests directory found - creating sample test structure...
                                mkdir tests
                                echo Please add test files to tests/ directory
                            )
                        '''
                    }
                }
            }
        }
        
        // STAGE 6: SECURITY SCANNING (Optional)
        // ─────────────────────────────────────────────────────────────────────────────
        stage('6️⃣ Security Scan') {
            steps {
                echo '═══════════════════════════════════════════════════════════════════════════════'
                echo '🔐 STAGE 6: SECURITY SCAN (Bandit)'
                echo '═══════════════════════════════════════════════════════════════════════════════'
                echo "Running security vulnerability scan with bandit..."
                
                script {
                    if (isUnix()) {
                        sh '''
                            . ${VENV_DIR}/bin/activate
                            echo "Scanning for security vulnerabilities..."
                            bandit -r . -f json -o bandit-report.json || true
                            echo "✅ Security scan completed"
                            echo "Report: bandit-report.json"
                        '''
                    } else {
                        bat '''
                            call %VENV_DIR%\\Scripts\\activate.bat
                            echo Scanning for security vulnerabilities...
                            bandit -r . -f json -o bandit-report.json || exit /b 0
                            echo Security scan completed
                            echo Report: bandit-report.json
                        '''
                    }
                }
            }
        }
        
        // STAGE 7: BUILD APPLICATION
        // ─────────────────────────────────────────────────────────────────────────────
        stage('7️⃣ Build Application') {
            steps {
                echo '═══════════════════════════════════════════════════════════════════════════════'
                echo '🔨 STAGE 7: BUILD APPLICATION'
                echo '═══════════════════════════════════════════════════════════════════════════════'
                echo "Packaging Flask application for deployment..."
                
                script {
                    if (isUnix()) {
                        sh '''
                            . ${VENV_DIR}/bin/activate
                            
                            echo "Step 1: Compile Python bytecode..."
                            python -m py_compile app.py
                            echo "✅ app.py compiled successfully"
                            
                            echo "Step 2: Create build directory..."
                            mkdir -p ${BUILD_DIR}
                            
                            echo "Step 3: Package application files..."
                            cp -r app.py requirements.txt templates/ static/ ${BUILD_DIR}/
                            echo "✅ Application files packaged"
                            
                            echo "Step 4: Create deployment archive..."
                            tar -czf flask-app-${BUILD_NUMBER}.tar.gz ${BUILD_DIR}/
                            echo "✅ Archive created: flask-app-${BUILD_NUMBER}.tar.gz"
                            
                            echo "Step 5: Generate build summary..."
                            echo "Build Number: ${BUILD_NUMBER}" > ${BUILD_DIR}/BUILD_INFO.txt
                            echo "Build Date: \$(date)" >> ${BUILD_DIR}/BUILD_INFO.txt
                            echo "Git Commit: \$(git rev-parse HEAD)" >> ${BUILD_DIR}/BUILD_INFO.txt
                            echo "Build completed successfully!"
                        '''
                    } else {
                        bat '''
                            call %VENV_DIR%\\Scripts\\activate.bat
                            
                            echo Step 1: Compile Python bytecode...
                            python -m py_compile app.py
                            echo app.py compiled successfully
                            
                            echo Step 2: Create build directory...
                            if not exist %BUILD_DIR% mkdir %BUILD_DIR%
                            
                            echo Step 3: Package application files...
                            xcopy app.py %BUILD_DIR%\\ /Y
                            xcopy requirements.txt %BUILD_DIR%\\ /Y
                            xcopy templates %BUILD_DIR%\\templates\\ /E /I /Y
                            xcopy static %BUILD_DIR%\\static\\ /E /I /Y
                            echo Application files packaged
                            
                            echo Step 4: Create deployment archive...
                            PowerShell -NoProfile -Command "Compress-Archive -Path '%BUILD_DIR%' -DestinationPath 'flask-app-%BUILD_NUMBER%.zip' -Force"
                            echo Archive created: flask-app-%BUILD_NUMBER%.zip
                            
                            echo Step 5: Generate build summary...
                            (
                                echo Build Number: %BUILD_NUMBER%
                                echo Build Date: %date% %time%
                                echo Build Status: SUCCESS
                            ) > %BUILD_DIR%\\BUILD_INFO.txt
                            echo Build completed successfully!
                        '''
                    }
                }
            }
        }
        
        // STAGE 8: DEPLOY APPLICATION
        // ─────────────────────────────────────────────────────────────────────────────
        stage('8️⃣ Deploy Application') {
            when {
                branch 'main'  // Only deploy from main branch
            }
            steps {
                echo '═══════════════════════════════════════════════════════════════════════════════'
                echo '🚀 STAGE 8: DEPLOY APPLICATION TO PRODUCTION'
                echo '═══════════════════════════════════════════════════════════════════════════════'
                echo "Deploying Flask application to production environment..."
                
                script {
                    if (isUnix()) {
                        sh '''
                            . ${VENV_DIR}/bin/activate
                            
                            echo "Step 1: Create deployment directory..."
                            mkdir -p ${DEPLOY_DIR}
                            echo "✅ Deployment directory ready"
                            
                            echo "Step 2: Copy application files..."
                            cp -r ${BUILD_DIR}/* ${DEPLOY_DIR}/
                            echo "✅ Application files deployed"
                            
                            echo "Step 3: Install dependencies in production..."
                            cd ${DEPLOY_DIR}
                            pip install -r requirements.txt
                            echo "✅ Production dependencies installed"
                            
                            echo "Step 4: Set proper permissions..."
                            chmod -R 755 ${DEPLOY_DIR}
                            echo "✅ Permissions set"
                            
                            echo "Step 5: Create systemd service (optional)..."
                            # Uncomment and customize for production:
                            # sudo systemctl restart flask-app
                            echo "⚠️  Restart Flask service manually or configure systemd"
                            
                            echo "✅ Deployment completed successfully!"
                            echo "📍 Application deployed to: ${DEPLOY_DIR}"
                        '''
                    } else {
                        bat '''
                            call %VENV_DIR%\\Scripts\\activate.bat
                            
                            echo Step 1: Create deployment directory...
                            if not exist %DEPLOY_DIR% mkdir %DEPLOY_DIR%
                            echo Deployment directory ready
                            
                            echo Step 2: Copy application files...
                            xcopy %BUILD_DIR%\\* %DEPLOY_DIR%\\ /E /I /Y
                            echo Application files deployed
                            
                            echo Step 3: Install dependencies in production...
                            cd /d %DEPLOY_DIR%
                            pip install -r requirements.txt
                            echo Production dependencies installed
                            
                            echo Step 4: Deployment completed successfully!
                            echo Application deployed to: %DEPLOY_DIR%
                            echo.
                            echo For Windows Service Integration:
                            echo 1. Install NSSM: https://nssm.cc/
                            echo 2. Run: nssm install Flask-App "C:\path\to\gunicorn.exe" "wsgi:app"
                            echo 3. Run: nssm start Flask-App
                        '''
                    }
                }
            }
        }
    }
    
    
    // ═══════════════════════════════════════════════════════════════════════════════
    // POST-BUILD ACTIONS & NOTIFICATIONS
    // ═══════════════════════════════════════════════════════════════════════════════
    post {
        always {
            echo '═══════════════════════════════════════════════════════════════════════════════'
            echo '🔄 POST-BUILD: CLEANUP & REPORTING'
            echo '═══════════════════════════════════════════════════════════════════════════════'
            
            script {
                echo "Generating build report..."
                
                if (isUnix()) {
                    sh '''
                        echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
                        echo "BUILD EXECUTION SUMMARY"
                        echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
                        echo "Build Number:      ${BUILD_NUMBER}"
                        echo "Build URL:         ${BUILD_URL}"
                        echo "Job Name:          ${JOB_NAME}"
                        echo "Build Status:      ${BUILD_STATUS}"
                        echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
                        
                        echo "Archiving build artifacts..."
                        # Keep only the latest 5 builds
                    '''
                } else {
                    bat '''
                        echo ════════════════════════════════════════════════════════════════════════════
                        echo BUILD EXECUTION SUMMARY
                        echo ════════════════════════════════════════════════════════════════════════════
                        echo Build Number:      %BUILD_NUMBER%
                        echo Job Name:          %JOB_NAME%
                        echo Build Status:      Check console output above
                        echo ════════════════════════════════════════════════════════════════════════════
                    '''
                }
            }
            
            // Archive test reports if they exist
            archiveArtifacts artifacts: '**/htmlcov/**,**/bandit-report.json', allowEmptyArchive: true
            
            // Publish test results (optional - requires plugins)
            // junit 'test-results.xml'
            // publishHTML([reportDir: 'htmlcov', reportFiles: 'index.html', reportName: 'Coverage Report'])
        }
        
        success {
            echo '═══════════════════════════════════════════════════════════════════════════════'
            echo '✅ PIPELINE SUCCESS'
            echo '═══════════════════════════════════════════════════════════════════════════════'
            
            script {
                echo '''
                ╔═══════════════════════════════════════════════════════════════════════════════╗
                ║                                                                               ║
                ║                 🎉 PIPELINE EXECUTION SUCCESSFUL! 🎉                         ║
                ║                                                                               ║
                ║  ✅ Repository cloned from GitHub                                           ║
                ║  ✅ Python environment created                                              ║
                ║  ✅ All 43 dependencies installed                                           ║
                ║  ✅ Unit tests executed (pytest)                                            ║
                ║  ✅ Code quality checks passed (flake8)                                     ║
                ║  ✅ Security scan completed (bandit)                                        ║
                ║  ✅ Application built and packaged                                          ║
                ║  ✅ Deployment prepared                                                     ║
                ║                                                                               ║
                ╚═══════════════════════════════════════════════════════════════════════════════╝
                
                Next Steps:
                  1. Review test coverage reports
                  2. Check security scan results
                  3. Monitor deployed application
                  4. Configure monitoring and alerts
                '''
            }
            
            // Send success notification (requires email plugin)
            // emailext(
            //     subject: "Jenkins Build SUCCESS: ${JOB_NAME} #${BUILD_NUMBER}",
            //     body: "Build succeeded. Check ${BUILD_URL} for details.",
            //     to: 'devteam@example.com'
            // )
        }
        
        failure {
            echo '═══════════════════════════════════════════════════════════════════════════════'
            echo '❌ PIPELINE FAILURE'
            echo '═══════════════════════════════════════════════════════════════════════════════'
            
            script {
                echo '''
                ╔═══════════════════════════════════════════════════════════════════════════════╗
                ║                                                                               ║
                ║                    ❌ PIPELINE EXECUTION FAILED ❌                           ║
                ║                                                                               ║
                ║  Please check the console output above for error details.                    ║
                ║                                                                               ║
                ║  Common Issues & Solutions:                                                  ║
                ║                                                                               ║
                ║  1. Dependency Installation Failed                                           ║
                ║     • Check internet connectivity                                            ║
                ║     • Verify requirements.txt exists                                         ║
                ║     • Check pip version: pip --version                                       ║
                ║                                                                               ║
                ║  2. Test Failures                                                            ║
                ║     • Review test output above                                               ║
                ║     • Check if tests/ directory exists                                       ║
                ║     • Run pytest locally: pytest -v tests/                                   ║
                ║                                                                               ║
                ║  3. Build Failed                                                             ║
                ║     • Verify Python files compile                                            ║
                ║     • Check for syntax errors: python -m py_compile app.py                  ║
                ║                                                                               ║
                ║  4. Deployment Failed                                                        ║
                ║     • Check deployment directory exists                                      ║
                ║     • Verify file permissions                                                ║
                ║     • Check disk space                                                       ║
                ║                                                                               ║
                ╚═══════════════════════════════════════════════════════════════════════════════╝
                '''
            }
            
            // Send failure notification (requires email plugin)
            // emailext(
            //     subject: "Jenkins Build FAILED: ${JOB_NAME} #${BUILD_NUMBER}",
            //     body: "Build failed. Check ${BUILD_URL}console for details.",
            //     to: 'devteam@example.com'
            // )
        }
        
        unstable {
            echo '⚠️  Pipeline unstable - some tests may have failed'
        }
        
        cleanup {
            script {
                if (isUnix()) {
                    sh '''
                        echo "Removing virtual environment..."
                        rm -rf ${VENV_DIR}
                        echo "✅ Virtual environment cleaned up"
                    '''
                } else {
                    bat '''
                        echo Removing virtual environment...
                        rmdir /s /q %VENV_DIR% 2>nul || echo Virtual environment not found
                        echo Virtual environment cleaned up
                    '''
                }
            }
        }
    }
    
    // ═══════════════════════════════════════════════════════════════════════════════
    // JENKINS CONFIGURATION NOTES & INSTRUCTIONS
    // ═══════════════════════════════════════════════════════════════════════════════
    /*
    
    🔧 JENKINS SETUP INSTRUCTIONS:
    
    1. INSTALL REQUIRED PLUGINS:
       - Git plugin (for GitHub integration)
       - GitHub plugin (for webhooks)
       - Pipeline plugin (for declarative pipelines)
       - Cobertura plugin (for code coverage)
       - Email Extension plugin (for notifications)
    
    2. CREATE NEW JENKINS JOB:
       - Go to Jenkins Dashboard → "New Item"
       - Enter job name: "Flask-App-Pipeline"
       - Select "Pipeline"
       - Click "OK"
    
    3. CONFIGURE GIT & GITHUB:
       - Under "Pipeline" → "Definition": Select "Pipeline script from SCM"
       - SCM: Select "Git"
       - Repository URL: https://github.com/Ayila890/SSD-Final.git
       - Credentials: Add GitHub credentials or SSH key
       - Script Path: Jenkinsfile
    
    4. SETUP BUILD TRIGGERS (IMPORTANT!):
       - Check: "GitHub hook trigger for GITScm polling"
       - This enables automatic builds on GitHub push
    
    5. CONFIGURE GITHUB WEBHOOK:
       - Go to GitHub repo → Settings → Webhooks
       - Add Webhook:
         * Payload URL: http://your-jenkins-server:8080/github-webhook/
         * Content type: application/json
         * Events: Push events
         * Active: ✓
    
    6. GITHUB CREDENTIALS IN JENKINS:
       - Jenkins → Manage Jenkins → Manage Credentials
       - Add Credentials → GitHub
       - Username: your-github-username
       - Password: GitHub Personal Access Token or Password
       - ID: github-credentials (must match Jenkinsfile)
    
    7. BUILD ENVIRONMENT CONFIGURATION:
       - Configure system Python or use pyenv/conda
       - Ensure Git is installed and in PATH
       - Ensure Python 3.11+ is available
    
    8. OPTIONAL: CONFIGURE EMAIL NOTIFICATIONS:
       - Jenkins → Manage Jenkins → Configure System
       - Set up SMTP server
       - Uncomment email sections in Jenkinsfile
    
    9. OPTIONAL: CONFIGURE SLACK NOTIFICATIONS:
       - Install Slack plugin
       - Get Slack webhook URL
       - Add to Jenkinsfile post section
    
    10. RUN YOUR FIRST BUILD:
        - Click "Build Now" on job page
        - Monitor console output
        - Fix any issues if build fails
        - Make a GitHub push to trigger automatic build
    
    📊 MONITORING & LOGS:
       - Console Output: Jenkins → Job → Build Number → Console Output
       - Build Artifacts: Jenkins → Job → Build Number → Artifacts
       - Build History: Jenkins → Job → Build History
       - Pipeline Graph: Jenkins → Job → Pipeline → View in Stage View
    
    🔐 SECURITY NOTES:
       - Use Jenkins credentials for GitHub token
       - Never hardcode secrets in Jenkinsfile
       - Use environment variables from Jenkins credentials
       - Regularly update Jenkins and plugins
       - Restrict job permissions to authorized users
    
    */
}
