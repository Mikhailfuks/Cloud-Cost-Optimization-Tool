package main

import ( context" ,flag" ,"fmt","log" ,"time"}
}

// Configuration for the cost optimization tool
type Config struct {
 Region  string
 BucketName string
 ReportFile string
}

// Function to get AWS configuration
func getAWSConfig(region string) (*aws.Config, error) {
 cfg, err := config.LoadDefaultConfig(context.TODO(), config.WithRegion(region))
 if err != nil {
  return nil, err
 }
 return &cfg, nil
}

// Function to get EC2 instance pricing information
func getEC2Pricing(cfg *aws.Config, instanceType string) (float64, error) {
 // Create a Pricing client
 pricingClient := pricing.NewFromConfig(*cfg)

 // Define the pricing request parameters
 params := &pricing.GetProductsInput{
  ServiceCode:  aws.String("AmazonEC2"),
  Filters: []types.Filter{
   {
    Type:  aws.String("TERM_MATCH"),
    Field: aws.String("instanceType"),
    Value: []string{instanceType},
   },
  },
 }

 // Get pricing information
 response, err := pricingClient.GetProducts(context.TODO(), params)
 if err != nil {
  return 0, err
 }

 // Extract the price from the response
 if len(response.PriceList) > 0 {
  return parseFloat(response.PriceList[0].Price.Amount), nil
 }
 return 0, fmt.Errorf("price not found for instance type: %s", instanceType)
}

// Function to get S3 storage cost
func getS3StorageCost(cfg *aws.Config, bucketName string, storageClass string) (float64, error) {
 // Create an S3 client
 s3Client := s3.NewFromConfig(*cfg)

 // Define the pricing request parameters
 params := &pricing.GetProductsInput{
  ServiceCode:  aws.String("AmazonS3"),
  Filters: []types.Filter{
   {
    Type:  aws.String("TERM_MATCH"),
    Field: aws.String("storageClass"),
    Value: []string{storageClass},
   },
  },
 }

 // Get pricing information
 response, err := pricingClient.GetProducts(context.TODO(), params)
 if err != nil {
  return 0, err
 }

 // Extract the price from the response
 if len(response.PriceList) > 0 {
  return parseFloat(response.PriceList[0].Price.Amount), nil
 }
 return 0, fmt.Errorf("price not found for storage class: %s", storageClass)
}

// Function to get EC2 instances in a given region
func getEC2Instances(cfg *aws.Config, region string) ([]*ec2.Instance, error) {
 // Create an EC2 client
 ec2Client := ec2.NewFromConfig(*cfg)
