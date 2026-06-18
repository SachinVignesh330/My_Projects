import json
import boto3
import os
from datetime import datetime

# Initialize the AWS SDK clients
iam = boto3.client('iam')
s3 = boto3.client('s3')
sns = boto3.client('sns')

# Pull configurations directly from your Lambda Environment Variables
AUDIT_BUCKET = os.environ['AUDIT_BUCKET']
SNS_TOPIC = os.environ['SNS_TOPIC_ARN']

# Custom compliance signature database of toxic permission combinations
DANGEROUS_COMBINATIONS = [
    {
        'name': 'EC2 instance launch with role passing',
        'description': 'Allows an attacker to deploy an EC2 server and pass an Admin profile to it to break confinement.',
        'required': ['iam:PassRole', 'ec2:RunInstances']
    },
    {
        'name': 'Direct policy attachment',
        'description': 'Allows an attacker to attach administrative managed policies directly to users or roles.',
        'required': ['iam:AttachRolePolicy']
    },
    {
        'name': 'Inline policy injection',
        'description': 'Allows an attacker to inject administrative inline policies straight into execution environments.',
        'required': ['iam:PutRolePolicy']
    }
]

def get_all_permissions(role_name):
    """Dynamically queries the live AWS IAM API to collect every permission block."""
    permissions = set()
    try:
        # Scan all attached AWS-managed or Customer-managed policies
        attached = iam.list_attached_role_policies(RoleName=role_name)
        for policy in attached['AttachedPolicies']:
            detail = iam.get_policy(PolicyArn=policy['PolicyArn'])
            vid = detail['Policy']['DefaultVersionId']
            version = iam.get_policy_version(PolicyArn=policy['PolicyArn'], VersionId=vid)
            statements = version['PolicyVersion']['Document'].get('Statement', [])
            if isinstance(statements, dict): 
                statements = [statements]
            for s in statements:
                if s.get('Effect') == 'Allow':
                    actions = s.get('Action', [])
                    if isinstance(actions, str): 
                        actions = [actions]
                    permissions.update(actions)
                    
        # Scan all custom inline policies embedded directly on the role profile
        inline = iam.list_role_policies(RoleName=role_name)
        for pname in inline['PolicyNames']:
            p = iam.get_role_policy(RoleName=role_name, PolicyName=pname)
            statements = p['PolicyDocument'].get('Statement', [])
            if isinstance(statements, dict): 
                statements = [statements]
            for s in statements:
                if s.get('Effect') == 'Allow':
                    actions = s.get('Action', [])
                    if isinstance(actions, str): 
                        actions = [actions]
                    permissions.update(actions)
    except Exception as e:
        print(f"Error compiling full policy mapping for target role {role_name}: {e}")
    return permissions

def check_escalation_paths(permissions):
    """Evaluates compiled permissions against our toxic signatures database."""
    findings = []
    for combo in DANGEROUS_COMBINATIONS:
        has_all = all(
            perm in permissions or perm.split(':')[0] + ':*' in permissions or '*' in permissions
            for perm in combo['required']
        )
        if has_all:
            findings.append(combo)
    return findings

def lambda_handler(event, context):
    """The serverless execution engine handler invoked by Amazon EventBridge."""
    print("Incoming Event Data Stream: ", json.dumps(event))
    detail = event.get('detail', {})
    event_name = detail.get('eventName', '')
    
    if event_name not in ['AttachRolePolicy', 'PutRolePolicy', 'CreateRole']:
        return {'statusCode': 200, 'body': 'Irrelevant mutation target event'}
        
    role_name = detail.get('requestParameters', {}).get('roleName')
    if not role_name:
        return {'statusCode': 200, 'body': 'Missing parameter field: roleName'}
        
    triggered_by = detail.get('userIdentity', {}).get('arn', 'unknown')
    
    permissions = get_all_permissions(role_name)
    findings = check_escalation_paths(permissions)
    
    timestamp = datetime.utcnow().isoformat()
    result = {
        'timestamp': timestamp,
        'role_name': role_name,
        'triggered_by': triggered_by,
        'findings': findings,
        'status': 'DANGEROUS' if findings else 'CLEAN'
    }
    
    # Save structured audit report into S3
    s3.put_object(
        Bucket=AUDIT_BUCKET,
        Key=f"findings/{timestamp[:10]}/{role_name}-{timestamp}.json",
        Body=json.dumps(result, indent=2)
    )
    
    # Dispatch a clean, human-readable alert via SNS if a violation was found
    if findings:
        formatted_findings = ""
        for i, finding in enumerate(findings, 1):
            formatted_findings += f"{i}. Alert Triggered: {finding['name']}\n"
            formatted_findings += f"   Description: {finding['description']}\n"
            formatted_findings += f"   Required Privileges Checked: {', '.join(finding['required'])}\n\n"

        sns.publish(
            TopicArn=SNS_TOPIC,
            Subject=f"SECURITY ALERT: Privilege Escalation on {role_name}",
            Message=(
                f" Privilege Escalation Signature Match Identified!\n\n"
                f"🔹 Target Role Altered: {role_name}\n"
                f"🔹 Actor Identity: {triggered_by}\n"
                f"🔹 Timestamp: {timestamp}\n\n"
                f"--------------------------------------------------\n"
                f" VIOLATION DETAILS:\n"
                f"--------------------------------------------------\n"
                f"{formatted_findings}"
                f"Please review this role change immediately in the IAM Console."
            )
        )
        
    return {'statusCode': 200, 'body': 'Security Analysis Engine Routine Clean Execution'}
